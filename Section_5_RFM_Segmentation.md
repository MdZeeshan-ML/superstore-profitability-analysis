# 👥 SECTION 5: RFM CUSTOMER SEGMENTATION

## Purpose
Segments customers into actionable groups based on purchase behavior (Recency, Frequency, Monetary value). Identifies which customers drive 80% of profit and which are at risk of churning.

---

## 📋 Cell Structure

### **Cell 7 - RFM Context (Markdown)**
Explains the RFM methodology and business impact

### **Cell 8 - RFM Calculation (Code)**
Calculates RFM scores and assigns customer segments

### **Cell 9 - Pareto Chart (Code)**
Visualizes which segments drive 80% of profit

### **Cell 10 - RFM Findings (Markdown)**
Summarizes key insights and action items

---

## 💻 Complete Code Templates

### Cell 7: RFM Context (Markdown)

```markdown
---

# 👥 SECTION 5: RFM CUSTOMER SEGMENTATION

## Why This Matters
80% of revenue comes from 20% of customers. RFM identifies WHO matters most.

## What is RFM?
- **R**ecency: Days since last purchase
- **F**requency: Number of orders
- **M**onetary: Total revenue

## Segments Created
- Champions: High R, F, M
- At Risk: Low R, high F, M (PRIORITY)
- Hibernating: Low R, F, M
- Lost: Very low R, F, M
```

---

### Cell 8: RFM Calculation (Code)

```python
# Calculate RFM
reference_date = df['Order_Date'].max() + pd.Timedelta(days=1)

rfm = df.groupby('Customer ID').agg({
    'Order_Date': lambda x: (reference_date - x.max()).days,
    'Order ID': 'nunique',
    'Sales': 'sum'
}).reset_index()

rfm.columns = ['Customer_ID', 'Recency', 'Frequency', 'Monetary']

# Score assignment (1-5)
rfm['R_Score'] = pd.qcut(rfm['Recency'], 5, labels=[5,4,3,2,1], duplicates='drop')
rfm['F_Score'] = pd.qcut(rfm['Frequency'].rank(method='first'), 5, labels=[1,2,3,4,5])
rfm['M_Score'] = pd.qcut(rfm['Monetary'], 5, labels=[1,2,3,4,5])

rfm['RFM_Score'] = rfm['R_Score'].astype(str) + rfm['F_Score'].astype(str) + rfm['M_Score'].astype(str)

# Segment mapping function
def get_segment(score):
    if score in ['555', '554', '544', '545']:
        return 'Champions'
    elif score in ['331', '321', '312', '221']:
        return 'At Risk'
    elif score in ['255', '254', '245']:
        return 'Hibernating'
    elif score in ['111', '112', '121']:
        return 'Lost'
    else:
        return 'Other'

rfm['Segment'] = rfm['RFM_Score'].apply(get_segment)

print(f"📊 Segmentation Complete: {len(rfm)} customers")
print(rfm['Segment'].value_counts())
```

---

### Cell 9: Pareto Chart (Code)

```python
# Merge with transactions
df_merged = df.merge(rfm[['Customer_ID', 'Segment']], 
                     left_on='Customer ID', right_on='Customer_ID')

# Aggregate by segment
segment_summary = df_merged.groupby('Segment').agg({
    'Customer_ID': 'nunique',
    'Sales': 'sum',
    'Profit': 'sum'
}).reset_index()

segment_summary = segment_summary.sort_values('Profit', ascending=False)
segment_summary['Cumulative_%'] = (segment_summary['Profit'].cumsum() / 
                                    segment_summary['Profit'].sum() * 100)

# Create Pareto chart
fig = make_subplots(specs=[[{"secondary_y": True}]])

fig.add_trace(
    go.Bar(x=segment_summary['Segment'], y=segment_summary['Profit'], 
           name='Profit', marker_color='#3498db'),
    secondary_y=False
)

fig.add_trace(
    go.Scatter(x=segment_summary['Segment'], y=segment_summary['Cumulative_%'],
               name='Cumulative %', mode='lines+markers', 
               marker_color='#e74c3c'),
    secondary_y=True
)

fig.add_hline(y=80, line_dash="dash", line_color="green", secondary_y=True)

fig.update_layout(title='📊 Pareto Analysis: Which Segments Drive 80% of Profit?',
                  height=500)
fig.show()
```

---

### Cell 10: RFM Findings (Markdown)

```markdown
## 🔍 RFM FINDINGS

### Critical Insights
1. **Top 4 segments** generate 80% of profit
2. **At Risk segment:** High value but churning - URGENT priority
3. **Champions:** Protect with VIP program
4. **Lost customers:** Don't waste resources

### Action Plan
- Focus 80% of retention budget on top 4 segments
- Launch win-back campaign for "At Risk" within 14 days
- Implement VIP perks for Champions

---
```

---

## 🔍 Deep Dive: RFM Methodology

### What is RFM?

RFM is a proven customer segmentation technique using three behavioral metrics:

#### Recency (R)
**Definition:** Days since last purchase  
**Calculation:** `(Today's Date - Last Order Date)`  
**Scoring:** Lower recency = Higher score  
- Score 5: Purchased within last 30 days
- Score 1: Hasn't purchased in 300+ days

**Business Logic:** Recent customers are more likely to buy again

#### Frequency (F)
**Definition:** Total number of purchases  
**Calculation:** `COUNT(DISTINCT Order_ID)`  
**Scoring:** Higher frequency = Higher score  
- Score 5: 10+ orders
- Score 1: 1-2 orders

**Business Logic:** Repeat customers are more valuable and loyal

#### Monetary (M)
**Definition:** Total revenue generated  
**Calculation:** `SUM(Sales)`  
**Scoring:** Higher spending = Higher score  
- Score 5: Spent $5,000+
- Score 1: Spent < $500

**Business Logic:** High spenders deserve premium treatment

---

## 🎯 Segment Definitions

### Champions (RFM: 555, 554, 544, 545)
**Profile:**
- Bought recently
- Buy frequently
- Spend the most

**Characteristics:**
- 15-20% of customer base
- 40-50% of revenue
- Highest lifetime value

**Strategy:**
- VIP loyalty program
- Early access to new products
- Personalized service
- Ask for referrals/reviews

---

### At Risk (RFM: 331, 321, 312, 221)
**Profile:**
- Haven't purchased recently (low R)
- Were frequent buyers (high F)
- Spent well in the past (high M)

**Characteristics:**
- 10-15% of customer base
- HIGH VALUE but CHURNING
- Critical intervention needed

**Strategy:**
- Win-back email campaign
- Special "we miss you" discount (20%)
- Phone call from account manager
- Survey to understand why they left

---

### Hibernating (RFM: 255, 254, 245)
**Profile:**
- Long time since last purchase
- Moderate past frequency
- Moderate past spending

**Characteristics:**
- 20-25% of customer base
- Potential for reactivation
- Lower priority than "At Risk"

**Strategy:**
- Automated email reactivation series
- Product recommendations based on past purchases
- Seasonal promotions

---

### Lost (RFM: 111, 112, 121)
**Profile:**
- Haven't purchased in very long time
- Only bought 1-2 times historically
- Low total spending

**Characteristics:**
- 15-20% of customer base
- Lowest recovery probability
- Not worth significant investment

**Strategy:**
- Minimal effort (low-cost email campaigns)
- Remove from expensive marketing channels
- Consider suppression to reduce costs

---

### Other (All remaining RFM combinations)
**Profile:** Mid-tier customers not in extreme segments

**Strategy:**
- Standard marketing campaigns
- Nurture toward Champion status
- Monitor for movement to At Risk

---

## 📊 Code Explanation

### Step 1: Calculate Reference Date
```python
reference_date = df['Order_Date'].max() + pd.Timedelta(days=1)
```
**Why +1 day?** Ensures most recent customers have Recency = 1 (not 0)

### Step 2: Aggregate by Customer
```python
rfm = df.groupby('Customer ID').agg({
    'Order_Date': lambda x: (reference_date - x.max()).days,
    'Order ID': 'nunique',
    'Sales': 'sum'
}).reset_index()
```
**Explanation:**
- `x.max()`: Most recent order date for each customer
- `nunique`: Count distinct orders (handles multiple line items)
- `sum`: Total revenue across all orders

### Step 3: Assign Scores (Quintile Method)
```python
rfm['R_Score'] = pd.qcut(rfm['Recency'], 5, labels=[5,4,3,2,1], duplicates='drop')
```
**Why `pd.qcut`?** Divides data into equal-sized buckets (quintiles)  
**Why reverse labels [5,4,3,2,1]?** Lower recency = better = higher score

**Alternative: Manual Thresholds**
```python
# Define your own cutoffs
rfm['R_Score'] = rfm['Recency'].apply(lambda x: 
    5 if x <= 30 else
    4 if x <= 90 else
    3 if x <= 180 else
    2 if x <= 365 else 1
)
```

### Step 4: Create Segment Logic
```python
def get_segment(score):
    if score in ['555', '554', '544', '545']:
        return 'Champions'
    # ...
```
**Customization:** Adjust score combinations based on your business needs

---

## 📊 Pareto Analysis Explained

### What is the 80/20 Rule?
80% of results come from 20% of efforts. In business:
- 80% of revenue from 20% of customers
- 80% of profit from 20% of products
- 80% of complaints from 20% of issues

### How the Chart Works

**Blue Bars:** Profit per segment (absolute $)  
**Red Line:** Cumulative profit percentage  
**Green Dashed Line:** 80% threshold  

**Interpretation:**
If the red line crosses 80% after 2 segments:
⇒ Focus 80% of your resources on those 2 segments

---

## 💡 Advanced Customizations

### 1. Add More Segments

```python
def get_segment_detailed(score):
    # Champions tier
    if score in ['555']:
        return 'Champions - Platinum'
    elif score in ['554', '544', '545']:
        return 'Champions - Gold'
    
    # Loyal customers
    elif score in ['444', '445', '454', '455']:
        return 'Loyal Customers'
    
    # New customers
    elif score in ['511', '512', '521']:
        return 'New Customers'
    
    # At risk tiers
    elif score in ['331', '321']:
        return 'At Risk - High Value'
    elif score in ['312', '221']:
        return 'At Risk - Medium Value'
    
    # Rest...
    else:
        return 'Other'
```

### 2. Weighted RFM Score

```python
# If you care more about Monetary than Frequency:
rfm['Weighted_Score'] = (
    rfm['R_Score'].astype(int) * 0.3 +
    rfm['F_Score'].astype(int) * 0.2 +
    rfm['M_Score'].astype(int) * 0.5
)
```

### 3. Dynamic Date Ranges

```python
# Only analyze last 12 months
recent_df = df[df['Order_Date'] >= df['Order_Date'].max() - pd.Timedelta(days=365)]
# Then run RFM on recent_df
```

### 4. Add Customer Details to Output

```python
# Merge customer names back
customer_names = df[['Customer ID', 'Customer Name']].drop_duplicates()
rfm_detailed = rfm.merge(customer_names, left_on='Customer_ID', right_on='Customer ID')

# Export Champions for sales team
champions = rfm_detailed[rfm_detailed['Segment'] == 'Champions']
champions.to_csv('Champions_List.csv', index=False)
print(f"Exported {len(champions)} Champions to CSV")
```

---

## 🚨 Common Issues & Solutions

### Issue 1: Too Many "Other" Customers
**Problem:** 60%+ customers in "Other" category  
**Solution:** Expand segment definitions to include more RFM combinations

### Issue 2: Unbalanced Segments
**Problem:** 90% customers are "Lost"  
**Solution:** Adjust quintile method to deciles (10 buckets) for more granularity

### Issue 3: No "At Risk" Customers
**Problem:** Business might be too new (no churn yet)  
**Solution:** Lower the Recency threshold (e.g., 60 days instead of 90)

---

## 📝 Real-World Application

### Monthly RFM Refresh Process

```python
# 1. Load latest data
df = pd.read_csv('latest_transactions.csv')

# 2. Run RFM calculation (use code from Cell 8)
# ...

# 3. Compare to last month
previous_rfm = pd.read_csv('rfm_last_month.csv')
changes = rfm.merge(previous_rfm[['Customer_ID', 'Segment']], 
                    on='Customer_ID', suffixes=('_new', '_old'))

# 4. Identify segment transitions
downgrades = changes[changes['Segment_new'] == 'At Risk'][
    changes['Segment_old'] == 'Champions'
]
print(f"⚠️ {len(downgrades)} Champions downgraded to At Risk!")

# 5. Alert sales team
if len(downgrades) > 0:
    downgrades.to_csv('urgent_follow_up.csv', index=False)
    # Send email notification
```

---

## ✅ Validation Checklist

Before proceeding:

- [ ] RFM scores calculated for all customers
- [ ] All scores are between 1-5
- [ ] At least 4 distinct segments created
- [ ] "Champions" segment exists and is < 30% of customers
- [ ] Pareto chart shows cumulative % reaching 80%
- [ ] Segment sizes seem reasonable (no single segment > 50%)

---

## 🎯 Next Steps

1. **Export segment lists** for marketing teams
2. **Set up automated campaigns** for each segment
3. **Proceed to Section 6:** Calculate Customer Lifetime Value per segment
4. **Track segment movement** month-over-month

---

**Section Status:** ✅ Template Complete  
**Dependencies:** Section 3 (Data Loading) must run first  
**Estimated Runtime:** 3-5 seconds  
**Output Type:** RFM DataFrame + Interactive Pareto chart
