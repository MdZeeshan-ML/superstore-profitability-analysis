# 📊 SECTION 8: NEW VS RETURNING CUSTOMER ANALYSIS

## Why This Matters
Understanding the balance between new customer acquisition and existing customer retention is critical for sustainable growth. This analysis reveals customer behavior patterns and guides marketing budget allocation.

---

## 🎯 Business Context

### Key Questions
- What percentage of revenue comes from repeat customers?
- How does average order value differ between new and returning customers?
- What is the repeat purchase rate?
- How long does it take for customers to make a second purchase?

### Strategic Impact
- **New customers** represent growth potential but have higher acquisition costs
- **Returning customers** are more profitable (no acquisition cost, higher trust)
- Industry benchmark: 60-70% of revenue should come from existing customers

---

## 📋 CELL 1: Analysis Context (Markdown)

```markdown
---

# 🔄 SECTION 8: NEW VS RETURNING CUSTOMER ANALYSIS

## Why This Matters
Acquiring a new customer costs 5-7x more than retaining an existing one. This section reveals whether we're investing in the right balance.

## Key Metrics
- **New Customer Rate:** % of first-time buyers
- **Repeat Purchase Rate:** % of customers who buy again
- **Revenue Split:** New vs Returning contribution
- **Time to Second Purchase:** Customer activation window

## Business Framework
```
Customer Lifecycle:
1. Acquisition (New) → High cost, low trust
2. Activation (First repeat) → Critical transition
3. Retention (Regular) → Low cost, high value
4. Growth (Upsell) → Maximum profitability
```
```

---

## 💻 CELL 2: Customer Classification (Code)

```python
# Classify customers as New vs Returning

# Get first purchase date for each customer
customer_first_purchase = df.groupby('Customer ID')['Order_Date'].min().reset_index()
customer_first_purchase.columns = ['Customer_ID', 'First_Purchase_Date']

# Merge back with original data
df_customer = df.merge(customer_first_purchase, left_on='Customer ID', right_on='Customer_ID')

# Classify each order
df_customer['Customer_Type'] = df_customer.apply(
    lambda row: 'New' if row['Order_Date'] == row['First_Purchase_Date'] else 'Returning',
    axis=1
)

# Customer-level summary
customer_summary = df.groupby('Customer ID').agg({
    'Order ID': 'nunique',  # Number of orders
    'Sales': 'sum',         # Total revenue
    'Profit': 'sum',        # Total profit
    'Order_Date': ['min', 'max']  # First and last purchase
}).reset_index()

customer_summary.columns = ['Customer_ID', 'Total_Orders', 'Total_Sales', 'Total_Profit', 'First_Order', 'Last_Order']
customer_summary['Customer_Type'] = customer_summary['Total_Orders'].apply(
    lambda x: 'One-Time' if x == 1 else 'Repeat'
)

print("✅ Customer classification complete")
print("\n📊 Customer Distribution:")
print(customer_summary['Customer_Type'].value_counts())
print(f"\n🔄 Repeat Rate: {(customer_summary['Customer_Type'] == 'Repeat').sum() / len(customer_summary) * 100:.1f}%")
```

### 🔍 What This Code Does

**Step-by-step breakdown:**
1. **First Purchase Identification:** Groups by Customer ID to find earliest order date
2. **Classification Logic:** Compares each order's date with customer's first purchase
3. **Customer-Level Metrics:** Aggregates total orders, revenue, and profit per customer
4. **Segmentation:** Splits customers into "One-Time" vs "Repeat"

**Business Logic:**
- **New:** First transaction ever with the company
- **Returning:** Any subsequent transaction
- **One-Time:** Customer who never came back (churn signal)
- **Repeat:** Customer with 2+ orders (successful retention)

**Expected Output:**
```
One-Time: ~60% of customers (industry typical: 55-70%)
Repeat: ~40% of customers
```

**Red Flags:**
- 80%+ One-Time customers → Retention problem, poor product-market fit
- 20%- One-Time customers → Unrealistic, check data quality

---

## 📈 CELL 3: Revenue and Profitability Comparison (Code)

```python
# Compare performance: New vs Returning at ORDER level
order_comparison = df_customer.groupby('Customer_Type').agg({
    'Order ID': 'count',
    'Sales': ['sum', 'mean'],
    'Profit': ['sum', 'mean'],
    'Quantity': 'mean'
}).round(2)

order_comparison.columns = ['Orders', 'Total_Revenue', 'Avg_Order_Value', 'Total_Profit', 'Avg_Profit', 'Avg_Items']
order_comparison['Revenue_%'] = (order_comparison['Total_Revenue'] / order_comparison['Total_Revenue'].sum() * 100).round(1)
order_comparison['Profit_%'] = (order_comparison['Total_Profit'] / order_comparison['Total_Profit'].sum() * 100).round(1)

print("📊 ORDER-LEVEL COMPARISON: New vs Returning")
print("="*70)
print(order_comparison)

# Compare at CUSTOMER level
customer_comparison = customer_summary.groupby('Customer_Type').agg({
    'Customer_ID': 'count',
    'Total_Sales': ['sum', 'mean'],
    'Total_Profit': ['sum', 'mean'],
    'Total_Orders': 'mean'
}).round(2)

customer_comparison.columns = ['Customers', 'Total_Revenue', 'Avg_Revenue_Per_Customer', 'Total_Profit', 'Avg_Profit_Per_Customer', 'Avg_Orders']

print("\n\n📊 CUSTOMER-LEVEL COMPARISON: One-Time vs Repeat")
print("="*70)
print(customer_comparison)

# Key insight calculation
repeat_customer_pct = (customer_summary['Customer_Type'] == 'Repeat').sum() / len(customer_summary) * 100
repeat_revenue_pct = (order_comparison.loc['Returning', 'Revenue_%'])

print(f"\n\n💡 KEY INSIGHT:")
print(f"🔹 {repeat_customer_pct:.1f}% of customers (Repeat) generate {repeat_revenue_pct:.1f}% of revenue")
print(f"🔹 Repeat customers are worth {customer_comparison.loc['Repeat', 'Avg_Revenue_Per_Customer'] / customer_comparison.loc['One-Time', 'Avg_Revenue_Per_Customer']:.1f}x more than one-time buyers")
```

### 🔍 What This Code Does

**Two-level analysis:**

1. **Order-Level View:** Shows performance of individual transactions
   - Do returning customers place larger orders?
   - Is profit margin better on repeat purchases?

2. **Customer-Level View:** Shows lifetime performance
   - How much is a repeat customer worth over time?
   - What's the multiplier effect of retention?

**Critical Metrics:**

| Metric | Meaning | Good Benchmark |
|--------|---------|----------------|
| Avg Order Value (Returning) | Basket size of loyal customers | 20-40% higher than new |
| Avg Profit (Returning) | Margin on repeat purchases | Should be higher (no discounts needed) |
| Avg Revenue Per Customer (Repeat) | Lifetime value multiplier | 3-5x one-time customers |

**Business Interpretation:**

```python
# Example expected insights:
# 1. Returning customers: Higher AOV (+25% typical)
#    → They know what they want, buy more confidently
# 
# 2. Returning customers: Better margins (+15% typical)
#    → Don't need discounts to convert
#
# 3. 40% of customers (repeat) → 65% of revenue
#    → Classic power law distribution
```

**Decision Framework:**
- If repeat customers = 3x+ value → Invest heavily in retention
- If AOV (returning) < AOV (new) → Investigate: Are we neglecting loyal customers?
- If profit margin is lower on returning → Over-discounting loyalty

---

## 📊 CELL 4: Visualization Dashboard (Code)

```python
# Create comprehensive comparison dashboard
fig = plt.figure(figsize=(18, 12))
gs = fig.add_gridspec(3, 3, hspace=0.3, wspace=0.3)

# 1. Customer Distribution Pie Chart
ax1 = fig.add_subplot(gs[0, 0])
customer_counts = customer_summary['Customer_Type'].value_counts()
colors = ['#e74c3c', '#2ecc71']
ax1.pie(customer_counts.values, labels=customer_counts.index, autopct='%1.1f%%', 
        colors=colors, startangle=90, textprops={'fontsize': 12, 'weight': 'bold'})
ax1.set_title('👥 Customer Distribution\n(One-Time vs Repeat)', fontsize=14, fontweight='bold')

# 2. Revenue Distribution Pie Chart
ax2 = fig.add_subplot(gs[0, 1])
revenue_by_type = df_customer.groupby('Customer_Type')['Sales'].sum()
ax2.pie(revenue_by_type.values, labels=revenue_by_type.index, autopct='%1.1f%%',
        colors=['#3498db', '#f39c12'], startangle=90, textprops={'fontsize': 12, 'weight': 'bold'})
ax2.set_title('💰 Revenue Distribution\n(New vs Returning Orders)', fontsize=14, fontweight='bold')

# 3. Profit Distribution Pie Chart
ax3 = fig.add_subplot(gs[0, 2])
profit_by_type = df_customer.groupby('Customer_Type')['Profit'].sum()
ax3.pie(profit_by_type.values, labels=profit_by_type.index, autopct='%1.1f%%',
        colors=['#9b59b6', '#1abc9c'], startangle=90, textprops={'fontsize': 12, 'weight': 'bold'})
ax3.set_title('📈 Profit Distribution\n(New vs Returning Orders)', fontsize=14, fontweight='bold')

# 4. Average Order Value Comparison
ax4 = fig.add_subplot(gs[1, 0])
aov_data = order_comparison['Avg_Order_Value']
bars = ax4.bar(aov_data.index, aov_data.values, color=['#3498db', '#f39c12'], edgecolor='black', linewidth=2)
ax4.set_title('💵 Average Order Value', fontsize=14, fontweight='bold')
ax4.set_ylabel('AOV ($)', fontsize=12)
for bar in bars:
    height = bar.get_height()
    ax4.text(bar.get_x() + bar.get_width()/2., height,
             f'${height:.0f}', ha='center', va='bottom', fontsize=12, fontweight='bold')
ax4.grid(axis='y', alpha=0.3)

# 5. Average Profit Comparison
ax5 = fig.add_subplot(gs[1, 1])
profit_data = order_comparison['Avg_Profit']
bars = ax5.bar(profit_data.index, profit_data.values, color=['#9b59b6', '#1abc9c'], edgecolor='black', linewidth=2)
ax5.set_title('📊 Average Profit per Order', fontsize=14, fontweight='bold')
ax5.set_ylabel('Profit ($)', fontsize=12)
for bar in bars:
    height = bar.get_height()
    ax5.text(bar.get_x() + bar.get_width()/2., height,
             f'${height:.0f}', ha='center', va='bottom', fontsize=12, fontweight='bold')
ax5.grid(axis='y', alpha=0.3)

# 6. Revenue per Customer Comparison
ax6 = fig.add_subplot(gs[1, 2])
cust_revenue = customer_comparison['Avg_Revenue_Per_Customer']
bars = ax6.bar(cust_revenue.index, cust_revenue.values, color=['#e74c3c', '#2ecc71'], edgecolor='black', linewidth=2)
ax6.set_title('🎯 Revenue per Customer (Lifetime)', fontsize=14, fontweight='bold')
ax6.set_ylabel('Revenue ($)', fontsize=12)
for bar in bars:
    height = bar.get_height()
    ax6.text(bar.get_x() + bar.get_width()/2., height,
             f'${height:.0f}', ha='center', va='bottom', fontsize=12, fontweight='bold')
ax6.grid(axis='y', alpha=0.3)

# 7. Time to Second Purchase Analysis
repeat_customers = customer_summary[customer_summary['Customer_Type'] == 'Repeat'].copy()
repeat_customers['Days_To_Second'] = (repeat_customers['Last_Order'] - repeat_customers['First_Order']).dt.days

ax7 = fig.add_subplot(gs[2, :2])
ax7.hist(repeat_customers['Days_To_Second'], bins=30, color='#3498db', edgecolor='black', alpha=0.7)
median_days = repeat_customers['Days_To_Second'].median()
ax7.axvline(median_days, color='red', linestyle='--', linewidth=2, label=f'Median: {median_days:.0f} days')
ax7.set_title('⏱️ Time Between First and Last Purchase (Repeat Customers)', fontsize=14, fontweight='bold')
ax7.set_xlabel('Days', fontsize=12)
ax7.set_ylabel('Number of Customers', fontsize=12)
ax7.legend(fontsize=11)
ax7.grid(axis='y', alpha=0.3)

# 8. Orders per Customer Distribution
ax8 = fig.add_subplot(gs[2, 2])
order_dist = customer_summary['Total_Orders'].value_counts().sort_index().head(10)
ax8.bar(order_dist.index, order_dist.values, color='#2ecc71', edgecolor='black')
ax8.set_title('📦 Orders per Customer Distribution', fontsize=14, fontweight='bold')
ax8.set_xlabel('Number of Orders', fontsize=12)
ax8.set_ylabel('Number of Customers', fontsize=12)
ax8.grid(axis='y', alpha=0.3)

plt.suptitle('🔄 NEW VS RETURNING CUSTOMER ANALYSIS DASHBOARD', fontsize=18, fontweight='bold', y=0.995)
plt.tight_layout()
plt.show()

print(f"\n📊 Median time to repeat purchase: {median_days:.0f} days")
print(f"🎯 Target: Launch reactivation campaign at {int(median_days * 0.8)} days")
```

### 🔍 What This Visualization Shows

**Dashboard Components:**

1. **Customer Distribution (Top Left)**
   - Shows one-time vs repeat customer count
   - Typical: 55-70% one-time, 30-45% repeat

2. **Revenue Distribution (Top Middle)**
   - Shows revenue split between new and returning orders
   - Goal: 60-70% from returning customers

3. **Profit Distribution (Top Right)**
   - Profit contribution by customer type
   - Should match or exceed revenue % (better margins)

4. **Average Order Value (Middle Left)**
   - Compares basket size
   - Returning should be 20-40% higher

5. **Average Profit (Middle Center)**
   - Margin comparison
   - Returning should have higher margins (no acquisition discounts)

6. **Revenue per Customer (Middle Right)**
   - Lifetime value multiplier
   - Shows power of retention

7. **Time to Second Purchase (Bottom Left)**
   - Customer activation timeline
   - Critical for win-back campaigns

8. **Orders per Customer (Bottom Right)**
   - Purchase frequency distribution
   - Identifies power users

**Business Insights to Extract:**
- If median days to repeat = 90, launch win-back at day 72 (80%)
- If 70% are one-time, focus on onboarding/first experience
- If returning AOV is lower, investigate loyalty program issues

---

## 💡 CELL 5: Findings and Recommendations (Markdown)

```markdown
## 🔍 KEY FINDINGS: NEW VS RETURNING ANALYSIS

### Critical Metrics
```
[Insert from your analysis]
- One-Time Customers: X% (Target: <65%)
- Repeat Purchase Rate: Y% (Target: >35%)
- Returning Revenue Share: Z% (Target: 60-70%)
- AOV Lift (Returning): +A% (Benchmark: +25%)
- Median Days to Repeat: B days
```

### Strategic Insights

#### 1️⃣ Revenue Concentration
- **Finding:** [X]% of revenue comes from returning customers ([Y]% of customer base)
- **Interpretation:** Strong retention = sustainable business model
- **Benchmark:** Healthy businesses see 60-70% revenue from existing customers

#### 2️⃣ Customer Value Multiplier
- **Finding:** Repeat customers worth [Z]x more than one-time buyers
- **Interpretation:** Each successful activation = [$X] additional lifetime value
- **Action:** Justify higher retention investment

#### 3️⃣ Purchase Patterns
- **Finding:** Median time to second purchase = [B] days
- **Critical Window:** Days 30-90 post-purchase
- **Churn Risk:** Customers who don't repurchase within [B×1.5] days likely lost

---

## 🎯 ACTION PLAN

### IMMEDIATE (0-30 days)

**1. Win-Back Campaign Automation**
- **Trigger:** Customer reaches 80% of median days-to-repeat without new order
- **Timing:** Launch at day [B × 0.8]
- **Offer:** 15% discount + personalized product recommendations
- **Expected Impact:** 10-15% reactivation rate = $[X] additional revenue

**2. First Purchase Follow-Up**
- **Timing:** Day 3, 7, 14, 30 post-purchase
- **Content:** 
  - Day 3: Thank you + product tips
  - Day 7: Related product suggestions
  - Day 14: Loyalty program invitation
  - Day 30: Exclusive discount (if no repeat purchase)
- **Goal:** Increase repeat rate from [Y]% to [Y+5]%

---

### SHORT-TERM (30-90 days)

**3. Loyalty Program Launch**
- **Target:** Customers with 2+ orders
- **Benefits:** 
  - 10% discount on 3rd purchase
  - Free shipping for members
  - Early access to sales
- **Investment:** $[X] setup + $[Y]/month
- **ROI:** Increase repeat customer value by [Z]%

**4. Personalization Engine**
- **Technology:** Email segmentation by purchase history
- **Segments:**
  - One-time buyers → Reactivation focus
  - 2-3 orders → Loyalty building
  - 4+ orders → VIP treatment
- **Expected Impact:** 20% increase in email conversion

---

### STRATEGIC (90+ days)

**5. Predictive Churn Model**
- **Data:** Purchase frequency, recency, AOV trends
- **Output:** Churn risk score per customer
- **Action:** Proactive outreach before churn occurs
- **Technology:** Python sklearn or commercial tool

**6. Referral Program**
- **Target:** Repeat customers only (proven product-market fit)
- **Incentive:** Give $20, Get $20
- **Logic:** Referred customers have 37% higher retention
- **Goal:** Turn best customers into acquisition channel

---

## 📊 SUCCESS METRICS

| Initiative | Metric | Current | Target | Timeline |
|------------|--------|---------|--------|----------|
| Win-Back Campaign | Reactivation Rate | [X]% | [X+10]% | 30 days |
| First Purchase Follow-Up | Repeat Rate | [Y]% | [Y+5]% | 60 days |
| Loyalty Program | Avg Orders (Repeat) | [Z] | [Z+1] | 90 days |
| Overall | Returning Revenue % | [A]% | 65% | 6 months |

---

## 🚫 WHAT NOT TO DO

**Don't over-discount:**
- One-time customers are not lost causes
- Focus on value communication, not just price

**Don't ignore the 80/20 rule:**
- 20% of customers drive 80% of profit
- Protect them at all costs

**Don't delay activation:**
- Customer habits form in first 30 days
- Waiting 60+ days = much lower conversion

---

## ✅ SECTION COMPLETE

**Next Analysis:** Geographic Performance (Section 9)

**Key Takeaway:** Retention is more profitable than acquisition. A 5% increase in repeat rate = [X]% profit increase.

---
```

---

## 📚 Technical Notes

### Data Requirements
- `Order_Date`: Must be datetime format
- `Customer ID`: Unique identifier (no nulls)
- `Sales`, `Profit`: Numeric values

### Assumptions
- First order date = acquisition date
- No external attribution data (all organic assumed)
- Refunds/returns not separately tracked

### Limitations
- Cannot distinguish between marketing-driven and organic repeats
- One-time customers may return after data cutoff
- Does not account for seasonality in repeat behavior

---

## 🔗 Related Sections
- **Section 5:** RFM Segmentation (deeper customer classification)
- **Section 6:** Customer Lifetime Value (monetary quantification)
- **Section 7:** Cohort Retention (time-based retention patterns)

---

**Status:** ✅ Complete | **Priority:** ⭐⭐ | **Business Impact:** High
