# 💵 SECTION 6: CUSTOMER LIFETIME VALUE (CLV)

## 📌 Purpose
Calculate the projected long-term revenue value of each customer to inform acquisition budgets, retention strategies, and VIP program targeting.

---

## 📋 Section Overview

| Cell | Type | Purpose | Output |
|------|------|---------|--------|
| 11 | Markdown | CLV Context & Methodology | Explanation |
| 12 | Code | CLV Calculation | Dataframe with CLV metrics |
| 13 | Code | CLV Distribution Visualization | Histogram + Top 10 Table |
| 14 | Markdown | CLV Findings & Actions | Business recommendations |

**Total Cells:** 4  
**Estimated Runtime:** ~10 seconds  
**Key Metrics:** Average CLV, 90th percentile, Median

---

## 🎨 CELL TEMPLATES

### **CELL 11: CLV Context (Markdown)**

```markdown
---

# 💵 SECTION 6: CUSTOMER LIFETIME VALUE (CLV)

## 🎯 Why This Matters
CLV tells you how much to invest in acquiring and retaining each customer. It's the foundation for:
- Setting customer acquisition cost (CAC) budgets
- Identifying which customers deserve premium service
- Calculating ROI on retention campaigns
- Prioritizing high-value segments

---

## 📊 Calculation Method

**Formula:**
```
CLV = (Average Order Value) × (Purchase Frequency) × (Customer Lifespan)
```

**3-Year Projection:**
```
CLV_3Y = AOV × (Orders/Year) × 3 years
```

---

## 🔍 What We're Measuring

| Metric | Definition | Business Use |
|--------|------------|--------------|
| **AOV** | Total Sales ÷ Number of Orders | Baseline revenue per transaction |
| **Purchase Rate** | Orders per Year | Loyalty indicator |
| **Lifespan** | Days between First & Last Order | Retention duration |
| **CLV_3Y** | 3-year projected value | Investment ceiling |

---

## 💡 Strategic Applications

1. **Acquisition Budget:** Max CAC = 30% of CLV
2. **VIP Program:** Target customers above 90th percentile
3. **Retention Priority:** Focus on customers at median+ CLV
4. **Segment Investment:** Allocate marketing budget proportional to CLV

---
```

---

### **CELL 12: CLV Calculation (Code)**

```python
# ============================================================================
# CELL 12: Customer Lifetime Value Calculation
# ============================================================================

# Calculate CLV components for each customer
customer_stats = df.groupby('Customer ID').agg({
    'Order ID': 'nunique',         # Total orders (Frequency)
    'Sales': 'sum',                # Total revenue (Monetary)
    'Order_Date': ['min', 'max']   # First and last purchase dates
}).reset_index()

# Flatten column names
customer_stats.columns = ['Customer_ID', 'Frequency', 'Total_Sales', 'First_Order', 'Last_Order']

# Calculate customer lifespan in days
customer_stats['Lifespan_Days'] = (customer_stats['Last_Order'] - customer_stats['First_Order']).dt.days

# Handle single-purchase customers (avoid division by zero)
customer_stats['Lifespan_Days'] = customer_stats['Lifespan_Days'].replace(0, 1)

# Calculate Average Order Value (AOV)
customer_stats['AOV'] = customer_stats['Total_Sales'] / customer_stats['Frequency']

# Calculate annual purchase rate
customer_stats['Purchase_Rate'] = customer_stats['Frequency'] / customer_stats['Lifespan_Days'] * 365

# Calculate 3-year CLV projection
customer_stats['CLV_3Y'] = customer_stats['AOV'] * customer_stats['Purchase_Rate'] * 3

# ============================================================================
# Summary Statistics
# ============================================================================

print("=" * 70)
print("💰 CUSTOMER LIFETIME VALUE SUMMARY")
print("=" * 70)

print(f"\n📊 Average CLV (3-year): ${customer_stats['CLV_3Y'].mean():,.2f}")
print(f"📈 Median CLV: ${customer_stats['CLV_3Y'].median():,.2f}")
print(f"🏆 90th Percentile CLV: ${customer_stats['CLV_3Y'].quantile(0.9):,.2f}")
print(f"⭐ Top 1% CLV: ${customer_stats['CLV_3Y'].quantile(0.99):,.2f}")

print(f"\n📉 Minimum CLV: ${customer_stats['CLV_3Y'].min():,.2f}")
print(f"📈 Maximum CLV: ${customer_stats['CLV_3Y'].max():,.2f}")

# Distribution quartiles
print(f"\n🔢 CLV Distribution by Quartile:")
print(f"   25th percentile: ${customer_stats['CLV_3Y'].quantile(0.25):,.2f}")
print(f"   50th percentile: ${customer_stats['CLV_3Y'].quantile(0.50):,.2f}")
print(f"   75th percentile: ${customer_stats['CLV_3Y'].quantile(0.75):,.2f}")

# Business insights
avg_cac_budget = customer_stats['CLV_3Y'].mean() * 0.3
print(f"\n💡 Recommended Max CAC (30% of avg CLV): ${avg_cac_budget:,.2f}")

print("=" * 70)
```

**🔍 Code Explanation:**

**Data Aggregation:**
- Groups transactions by `Customer ID` to calculate per-customer metrics
- `'Order ID': 'nunique'` counts distinct orders (frequency)
- `'Sales': 'sum'` totals lifetime revenue
- `'Order_Date': ['min', 'max']` captures first and last purchase dates

**Lifespan Calculation:**
- Subtracts first order from last order to get active days
- Replaces 0-day lifespans (single purchases) with 1 to avoid division errors

**Key Metrics:**
- **AOV**: Average revenue per order (`Total_Sales / Frequency`)
- **Purchase Rate**: Annual order frequency (`Frequency / Lifespan_Days * 365`)
- **CLV_3Y**: 3-year projected value (`AOV × Purchase_Rate × 3`)

**Statistical Summary:**
- Mean/median for central tendency
- 90th percentile identifies top-tier customers
- Quartiles show value distribution across customer base

---

### **CELL 13: CLV Visualization (Code)**

```python
# ============================================================================
# CELL 13: CLV Distribution Visualization
# ============================================================================

# Create histogram with Plotly
fig = px.histogram(
    customer_stats, 
    x='CLV_3Y', 
    nbins=50,
    title='📊 Customer Lifetime Value Distribution (3-Year Projection)',
    labels={'CLV_3Y': 'CLV ($)', 'count': 'Number of Customers'},
    color_discrete_sequence=['#3498db']
)

# Add median line
fig.add_vline(
    x=customer_stats['CLV_3Y'].median(), 
    line_dash="dash", 
    line_color="red",
    annotation_text=f"Median: ${customer_stats['CLV_3Y'].median():,.0f}",
    annotation_position="top"
)

# Add 90th percentile line
fig.add_vline(
    x=customer_stats['CLV_3Y'].quantile(0.9), 
    line_dash="dot", 
    line_color="green",
    annotation_text=f"Top 10%: ${customer_stats['CLV_3Y'].quantile(0.9):,.0f}",
    annotation_position="top"
)

fig.update_layout(
    height=500,
    showlegend=False,
    xaxis_title="Customer Lifetime Value ($)",
    yaxis_title="Number of Customers"
)

fig.show()

# ============================================================================
# Top Customers by CLV
# ============================================================================

print("\n" + "=" * 100)
print("⭐ TOP 10 CUSTOMERS BY LIFETIME VALUE")
print("=" * 100)

top_clv = customer_stats.nlargest(10, 'CLV_3Y')[
    ['Customer_ID', 'CLV_3Y', 'AOV', 'Frequency', 'Purchase_Rate']
].copy()

# Format for display
top_clv['CLV_3Y'] = top_clv['CLV_3Y'].apply(lambda x: f"${x:,.2f}")
top_clv['AOV'] = top_clv['AOV'].apply(lambda x: f"${x:,.2f}")
top_clv['Purchase_Rate'] = top_clv['Purchase_Rate'].apply(lambda x: f"{x:.2f} orders/year")

top_clv.columns = ['Customer ID', '3-Year CLV', 'Avg Order Value', 'Total Orders', 'Annual Order Rate']
top_clv.index = range(1, 11)

print(top_clv.to_string())
print("=" * 100)

# ============================================================================
# Segment customers by CLV tier
# ============================================================================

def categorize_clv(clv, median, p90):
    if clv >= p90:
        return 'Platinum'
    elif clv >= median:
        return 'Gold'
    else:
        return 'Silver'

customer_stats['CLV_Tier'] = customer_stats['CLV_3Y'].apply(
    lambda x: categorize_clv(x, customer_stats['CLV_3Y'].median(), customer_stats['CLV_3Y'].quantile(0.9))
)

tier_summary = customer_stats.groupby('CLV_Tier').agg({
    'Customer_ID': 'count',
    'CLV_3Y': 'mean',
    'Total_Sales': 'sum'
}).round(2)

tier_summary.columns = ['Customer Count', 'Avg CLV', 'Total Historical Sales']

print("\n📊 CLV TIER BREAKDOWN:")
print(tier_summary)
```

**🔍 Code Explanation:**

**Histogram Visualization:**
- Uses Plotly's `px.histogram` for interactive chart
- `nbins=50` creates 50 value buckets for smooth distribution
- Vertical lines mark median (typical customer) and 90th percentile (VIP threshold)

**Top 10 Table:**
- `nlargest(10, 'CLV_3Y')` selects highest-value customers
- Formats currency and rates for readability
- Provides actionable list for VIP targeting

**CLV Tiering:**
- **Platinum** (Top 10%): Premium service candidates
- **Gold** (Median+): Standard retention focus
- **Silver** (Below median): Basic service tier
- Enables segmented marketing and support strategies

**Business Use:**
- Identify customers worth personalized attention
- Quantify revenue concentration in top tier
- Set service level expectations by segment

---

### **CELL 14: CLV Findings (Markdown)**

```markdown
---

## 🔍 KEY CLV FINDINGS & STRATEGIC ACTIONS

### 📊 Distribution Insights

**Typical Customer Value:**
- Median CLV represents the "average" customer lifetime value
- 50% of customers fall below this threshold
- Use as baseline for standard acquisition costs

**High-Value Segment:**
- Top 10% (90th percentile+) are VIP candidates
- These customers justify premium service investments
- Should receive personalized retention campaigns

**Revenue Concentration:**
- Check if Platinum tier contributes >50% of total sales
- Indicates dependency on small customer group
- Requires protective retention strategies

---

### 💡 Strategic Action Plan

#### 1️⃣ Customer Acquisition Budget
**Rule:** Max CAC = 30% of Average CLV

**Calculation Example:**
- If Average CLV = $3,000
- Max CAC = $900 per customer
- Any acquisition cost above this erodes profitability

**Action:** Set hard cap on marketing spend per lead based on this threshold.

---

#### 2️⃣ VIP Program Design
**Target:** Customers with CLV ≥ 90th Percentile

**Benefits to Offer:**
- Free expedited shipping
- Dedicated customer success manager
- Early access to new products
- Exclusive discounts (10-15% vs. public promotions)

**Investment:** Budget 5-10% of Platinum tier CLV for retention perks

**Expected ROI:** 20-30% increase in retention rate = 2-3x return

---

#### 3️⃣ Retention Priority Matrix

| CLV Tier | Focus | Budget Allocation | Tactics |
|----------|-------|-------------------|---------|
| **Platinum** | Protect & Grow | 50% | Personal outreach, VIP perks |
| **Gold** | Maintain | 30% | Automated campaigns, loyalty points |
| **Silver** | Efficient Service | 20% | Self-service, occasional promotions |

**Implementation:**
- Assign account managers to Platinum customers
- Create automated email sequences for Gold tier
- Optimize support costs for Silver tier

---

#### 4️⃣ Upsell Strategy
**Target:** Gold tier customers approaching Platinum threshold

**Approach:**
- Identify Gold customers within $500 of Platinum minimum
- Offer bundle deals to increase AOV
- Create "path to VIP" incentive program

**Goal:** Convert 10-15% of top Gold customers to Platinum annually

---

### ⚠️ Risk Mitigation

**Over-Dependence on Top Tier:**
- If Platinum tier = >60% of revenue, business is vulnerable
- Diversification strategy needed to grow Gold tier base

**Churn in Platinum Tier:**
- Monitor for decreased order frequency
- Trigger immediate intervention if 60+ days inactive
- Assign retention specialist for personal outreach

**Low CLV Acquisition:**
- If new customer CLV trending below historical median, review:
  - Lead quality from marketing channels
  - Onboarding experience
  - Product-market fit

---

### 📈 Success Metrics to Track

| Metric | Current | Target (6 months) | Tracking Frequency |
|--------|---------|-------------------|-------------------|
| Avg CLV (All) | [From Cell 12] | +15% | Monthly |
| Platinum Tier % | [Calculate] | 12-15% | Quarterly |
| Gold→Platinum Conversion | N/A | 10% | Quarterly |
| Platinum Churn Rate | [Calculate] | <5% | Monthly |

---

### 🎯 Next Steps

1. **Week 1:** Export Platinum tier customer list → CRM
2. **Week 2:** Design VIP program benefits (collaborate with ops team)
3. **Week 3:** Launch Platinum retention campaign
4. **Week 4:** Set up CLV tracking dashboard in analytics tool

---

**📊 Continue to Section 7 for Cohort Retention Analysis →**

---
```

**🔍 Explanation of Findings:**

**Business Context:**
- Connects CLV calculations to real-world decisions
- Provides specific budget allocation rules (30% CAC rule)
- Defines customer tiers with clear service levels

**Actionable Framework:**
- VIP program design with measurable benefits
- Retention priority matrix allocates resources efficiently
- Upsell strategy targets high-potential customers

**Risk Management:**
- Identifies over-concentration risk (>60% from Platinum)
- Sets early warning triggers (60-day inactivity)
- Monitors acquisition quality through new customer CLV trends

**Implementation Roadmap:**
- 4-week plan with clear ownership
- Specific metrics to track progress
- Quarterly and monthly review cadence

---

## 🎯 SECTION 6 COMPLETION CHECKLIST

- [x] Cell 11: CLV methodology explained
- [x] Cell 12: Calculation code with summary stats
- [x] Cell 13: Distribution histogram + top 10 table + tier breakdown
- [x] Cell 14: Strategic findings with budget rules & action plan

**Expected Output:**
- CLV metrics for all customers
- Visual distribution with median/90th percentile markers
- Top 10 customer list for VIP targeting
- 3-tier segmentation (Platinum/Gold/Silver)
- CAC budget ceiling calculation
- 4-week implementation roadmap

---

**📂 File Location:** `Section_6_Customer_Lifetime_Value.md`  
**🔗 Next:** Section 7 - Cohort Retention Analysis
