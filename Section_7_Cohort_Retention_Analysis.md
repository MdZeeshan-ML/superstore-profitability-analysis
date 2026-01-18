# 📆 SECTION 7: COHORT RETENTION ANALYSIS

## 📌 Purpose
Track customer retention patterns over time by grouping customers based on their first purchase month (cohorts) to identify when customers churn and which acquisition periods performed best.

---

## 📋 Section Overview

| Cell | Type | Purpose | Output |
|------|------|---------|--------|
| 15 | Markdown | Cohort Context & Methodology | Explanation |
| 16 | Code | Cohort Data Preparation | Cohort pivot table |
| 17 | Code | Retention Heatmap | Visual heatmap + stats |
| 18 | Markdown | Cohort Findings & Actions | Business recommendations |

**Total Cells:** 4  
**Estimated Runtime:** ~15 seconds  
**Key Visualization:** Retention heatmap showing % of customers returning each month

---

## 🎨 CELL TEMPLATES

### **CELL 15: Cohort Context (Markdown)**

```markdown
---

# 📆 SECTION 7: COHORT RETENTION ANALYSIS

## 🎯 Why This Matters
Cohort analysis reveals **WHEN** customers churn and **WHICH** acquisition periods produced the best long-term customers. This helps:

- **Identify churn patterns:** Month 3? Month 6? Where do we lose them?
- **Evaluate marketing campaigns:** Did Q4 2015 customers stick around?
- **Design intervention timing:** When should win-back campaigns trigger?
- **Forecast revenue:** Predict future retention based on historical patterns

---

## 📊 What is Cohort Analysis?

**Cohort:** Group of customers who made their **first purchase** in the same month

**Retention Rate:** % of cohort members who return in subsequent months

**Example:**
- 100 customers acquired in Jan 2015 (Cohort)
- 40 made a purchase in Feb 2015 → 40% Month 1 retention
- 25 made a purchase in Mar 2015 → 25% Month 2 retention

---

## 🔍 How to Read the Heatmap

### Heatmap Structure
- **Rows:** Cohort (first purchase month)
- **Columns:** Months since first purchase (0, 1, 2, 3...)
- **Values:** % of original cohort still active

### Color Coding
- 🟢 **Green (80-100%):** Excellent retention
- 🟡 **Yellow (40-80%):** Normal retention
- 🔴 **Red (0-40%):** Poor retention - investigate!

### Key Patterns to Look For
1. **Steep drop in Month 1:** Onboarding issue
2. **Consistent decline:** Natural churn (expected)
3. **Sudden cliff at Month X:** Identify trigger event
4. **High Month 6+ retention:** Loyal customer base

---

## 💡 Business Questions We'll Answer

1. What % of customers return for a 2nd purchase?
2. At which month does retention stabilize?
3. Which cohorts (acquisition periods) had best retention?
4. Is there a "churn cliff" we need to address?

---
```

---

### **CELL 16: Cohort Preparation (Code)**

```python
# ============================================================================
# CELL 16: Cohort Data Preparation
# ============================================================================

# Create order month column
df['OrderMonth'] = df['Order_Date'].dt.to_period('M')

# Get first purchase month for each customer (defines their cohort)
cohort_data = df.groupby('Customer ID')['OrderMonth'].min().reset_index()
cohort_data.columns = ['Customer_ID', 'CohortMonth']

print("✅ Cohort assignment complete")
print(f"📅 First cohort: {cohort_data['CohortMonth'].min()}")
print(f"📅 Last cohort: {cohort_data['CohortMonth'].max()}")
print(f"👥 Total cohorts: {cohort_data['CohortMonth'].nunique()} months\n")

# Merge cohort month back to main dataframe
df_cohort = df.merge(cohort_data, left_on='Customer ID', right_on='Customer_ID')

# Calculate months since cohort (CohortIndex)
# CohortIndex = 0 means first purchase month
# CohortIndex = 1 means 1 month after first purchase, etc.
df_cohort['CohortIndex'] = (df_cohort['OrderMonth'] - df_cohort['CohortMonth']).apply(lambda x: x.n)

# ============================================================================
# Create Cohort Pivot Table
# ============================================================================

# Count unique customers per cohort-month combination
cohort_counts = df_cohort.groupby(['CohortMonth', 'CohortIndex'])['Customer_ID'].nunique().reset_index()

# Pivot to create cohort matrix
cohort_pivot = cohort_counts.pivot(
    index='CohortMonth', 
    columns='CohortIndex', 
    values='Customer_ID'
)

print("📊 COHORT MATRIX (Customer Counts):")
print(cohort_pivot.head(10))

# ============================================================================
# Calculate Retention Rates (% of original cohort)
# ============================================================================

# Get cohort size (Month 0 count)
cohort_size = cohort_pivot.iloc[:, 0]

# Calculate retention rate for each month
retention = cohort_pivot.divide(cohort_size, axis=0) * 100

print(f"\n📈 RETENTION RATES (% of Cohort):")
print(retention.head(10).round(1))

# ============================================================================
# Overall Retention Statistics
# ============================================================================

avg_retention = retention.mean(axis=0)

print("\n" + "="*70)
print("📏 AVERAGE RETENTION BY MONTH")
print("="*70)

for month_idx in range(min(13, len(avg_retention))):
    print(f"Month {month_idx:2d}: {avg_retention.iloc[month_idx]:5.1f}% of cohort active")

print("="*70)

# Key insights
if len(avg_retention) > 1:
    month_1_retention = avg_retention.iloc[1]
    print(f"\n⚠️  CRITICAL METRIC - Month 1 Retention: {month_1_retention:.1f}%")
    print(f"    → {100 - month_1_retention:.1f}% of customers never return after first purchase")

if len(avg_retention) > 6:
    month_6_retention = avg_retention.iloc[6]
    print(f"\n🎯 Long-Term Viability - Month 6 Retention: {month_6_retention:.1f}%")
    print(f"    → {month_6_retention:.1f}% of customers still active after 6 months")

print("\n")
```

**🔍 Code Explanation:**

**Cohort Assignment:**
- `df['OrderMonth'].dt.to_period('M')` converts dates to month periods (e.g., "2015-01")
- `groupby('Customer ID').min()` finds each customer's first purchase month
- This creates the cohort label for each customer

**Cohort Index Calculation:**
- `.apply(lambda x: x.n)` extracts the number of months between order and cohort
- CohortIndex = 0: First purchase
- CohortIndex = 1: One month after first purchase
- CohortIndex = 2: Two months after, etc.

**Pivot Table:**
- Rows = Cohorts (acquisition months)
- Columns = CohortIndex (months since acquisition)
- Values = Count of unique customers

**Retention Calculation:**
- `cohort_pivot.divide(cohort_size, axis=0)` divides each month by Month 0 count
- Multiplied by 100 to get percentage
- Shows what % of original cohort is still active

**Key Metrics:**
- **Month 1 Retention:** Critical for onboarding effectiveness
- **Month 6 Retention:** Indicator of long-term loyalty
- **Average retention curve:** Overall retention pattern

---

### **CELL 17: Retention Heatmap (Code)**

```python
# ============================================================================
# CELL 17: Cohort Retention Heatmap Visualization
# ============================================================================

# Create large figure for detailed heatmap
plt.figure(figsize=(18, 12))

# Create heatmap with retention percentages
sns.heatmap(
    retention, 
    annot=True,          # Show values in cells
    fmt='.0f',           # Format as integer (no decimals)
    cmap='RdYlGn',       # Red-Yellow-Green color scale
    vmin=0,              # Minimum value (0%)
    vmax=100,            # Maximum value (100%)
    linewidths=0.5,      # Grid lines
    linecolor='gray',    # Grid line color
    cbar_kws={'label': 'Retention Rate (%)'}
)

plt.title(
    '🔥 COHORT RETENTION HEATMAP - % of Customers Active by Month', 
    fontsize=18, 
    fontweight='bold',
    pad=20
)
plt.xlabel('Months Since First Purchase', fontsize=14, fontweight='bold')
plt.ylabel('Cohort (First Purchase Month)', fontsize=14, fontweight='bold')

# Rotate x-axis labels for readability
plt.xticks(rotation=0)
plt.yticks(rotation=0)

plt.tight_layout()
plt.show()

# ============================================================================
# Retention Curve Visualization
# ============================================================================

fig, ax = plt.subplots(figsize=(14, 7))

# Plot average retention curve
avg_retention_curve = retention.mean(axis=0)
ax.plot(
    avg_retention_curve.index, 
    avg_retention_curve.values,
    marker='o', 
    linewidth=3, 
    markersize=8,
    color='#3498db',
    label='Average Retention'
)

# Fill area under curve
ax.fill_between(
    avg_retention_curve.index,
    avg_retention_curve.values,
    alpha=0.3,
    color='#3498db'
)

# Add benchmark lines
ax.axhline(y=50, color='orange', linestyle='--', linewidth=2, alpha=0.7, label='50% Threshold')
ax.axhline(y=25, color='red', linestyle='--', linewidth=2, alpha=0.7, label='25% Critical Level')

ax.set_xlabel('Months Since First Purchase', fontsize=12, fontweight='bold')
ax.set_ylabel('Retention Rate (%)', fontsize=12, fontweight='bold')
ax.set_title('📉 Average Retention Curve Over Time', fontsize=16, fontweight='bold')
ax.legend(loc='upper right', fontsize=11)
ax.grid(alpha=0.3)

plt.tight_layout()
plt.show()

# ============================================================================
# Identify Best and Worst Cohorts
# ============================================================================

print("\n" + "="*70)
print("🏆 BEST PERFORMING COHORTS (6-Month Retention)")
print("="*70)

if 6 in retention.columns:
    best_cohorts = retention[6].nlargest(5).sort_values(ascending=False)
    for cohort, rate in best_cohorts.items():
        print(f"  {cohort}: {rate:.1f}% retention at Month 6")
else:
    print("  Insufficient data for 6-month retention")

print("\n" + "="*70)
print("⚠️  WORST PERFORMING COHORTS (6-Month Retention)")
print("="*70)

if 6 in retention.columns:
    worst_cohorts = retention[6].nsmallest(5).sort_values(ascending=True)
    for cohort, rate in worst_cohorts.items():
        print(f"  {cohort}: {rate:.1f}% retention at Month 6")
else:
    print("  Insufficient data for 6-month retention")

print("="*70)

# ============================================================================
# Churn Analysis
# ============================================================================

print("\n📉 CHURN RATE ANALYSIS (% Lost Each Month):\n")

churn_rates = -retention.diff(axis=1).iloc[:, 1:]
avg_churn = churn_rates.mean(axis=0)

for month_idx in range(min(12, len(avg_churn))):
    churn_val = avg_churn.iloc[month_idx]
    print(f"  Month {month_idx} → {month_idx+1}: {churn_val:.1f}% churn")

# Identify biggest churn cliff
if len(avg_churn) > 0:
    biggest_churn_month = avg_churn.idxmax()
    biggest_churn_rate = avg_churn.max()
    print(f"\n⚠️  CHURN CLIFF: Highest drop at Month {biggest_churn_month} ({biggest_churn_rate:.1f}% lost)")
```

**🔍 Code Explanation:**

**Heatmap Configuration:**
- `annot=True` displays retention % in each cell
- `cmap='RdYlGn'` creates intuitive color scheme (red=bad, green=good)
- `vmin=0, vmax=100` sets scale from 0% to 100%
- Larger figure size (18x12) ensures readability

**Retention Curve:**
- Line chart shows average retention across all cohorts
- Filled area visualizes retention "strength"
- Benchmark lines at 50% and 25% for context

**Best/Worst Cohorts:**
- Identifies which acquisition months produced most loyal customers
- Uses 6-month retention as quality benchmark
- Helps evaluate marketing campaign effectiveness

**Churn Analysis:**
- `.diff(axis=1)` calculates month-over-month change
- Negative diff = churn (customers lost)
- Identifies where biggest drop-off occurs

**Business Use:**
- **Churn cliff** = trigger point for intervention campaigns
- **Best cohorts** = replicate acquisition strategies
- **Worst cohorts** = avoid or fix those tactics

---

### **CELL 18: Cohort Findings (Markdown)**

```markdown
---

## 🔍 KEY COHORT FINDINGS & STRATEGIC ACTIONS

### 📊 Retention Pattern Insights

#### Month 1 Retention (Critical Metric)
**What It Means:**
- % of customers who return after first purchase
- Reflects onboarding experience quality
- Industry benchmark: 20-30% is typical

**If Below 20%:**
- ⚠️ Onboarding problem
- First purchase experience disappointing
- Expectations not met

**Action Plan:**
- Review first-time customer journey
- Implement post-purchase email sequence
- Offer "2nd purchase" incentive (15% off within 30 days)

---

#### Month 6 Retention (Long-Term Viability)
**What It Means:**
- % still active after half a year
- Indicator of true customer loyalty
- Industry benchmark: 10-20% is healthy

**If Above 15%:**
- ✅ Strong product-market fit
- Effective retention mechanisms
- Loyal customer base forming

**If Below 10%:**
- ⚠️ Weak long-term retention
- Need loyalty program
- Product may not create habit

---

#### Churn Cliff Identification
**What to Look For:**
- Month where biggest % drop occurs
- Often at Month 2-4 in retail

**Common Causes:**
| Churn Month | Likely Reason | Fix |
|-------------|---------------|-----|
| Month 1-2 | Poor onboarding | Welcome sequence, 2nd purchase offer |
| Month 3-4 | Product dissatisfaction | Quality check, loyalty program |
| Month 6+ | Lack of engagement | Re-engagement campaign, VIP perks |

---

### 🏆 Best vs. Worst Cohorts

**Why Some Cohorts Perform Better:**
1. **Seasonality:** Holiday shoppers vs. regular traffic
2. **Marketing Quality:** Targeted campaigns vs. broad ads
3. **Product Mix:** Best-sellers available vs. clearance focus
4. **Onboarding:** Strong welcome sequence vs. none

**Action:**
- Compare marketing spend/strategy for best vs. worst cohorts
- Replicate tactics from high-retention months
- Avoid acquisition channels that produced worst cohorts

---

### 💡 Strategic Action Plan

#### 1️⃣ Immediate: Fix Month 1 Retention
**Goal:** Increase 2nd purchase rate by 30%

**Tactics:**
```
Day 3 Post-Purchase:
  → Send "How to get the most from [product]" email
  → Include customer success tips
  
 Day 7:
  → Request feedback survey
  → Offer 15% off next purchase for completion
  
Day 21:
  → "We miss you" email if no 2nd order
  → Highlight best-sellers, time-limited discount
```

**Investment:** $500 email automation setup
**Expected ROI:** 25-30% Month 1 retention increase = $[X] annual value

---

#### 2️⃣ Short-Term: Address Churn Cliff
**Trigger:** When customer reaches [X months] since last order

**Intervention:**
- Personalized "comeback" email with 20% incentive
- Free shipping offer
- Showcase new products since their last visit

**Automation:**
- Set up in CRM/email platform
- A/B test subject lines and offers
- Track win-back rate (target: 10-15%)

---

#### 3️⃣ Long-Term: Build Loyalty Program
**Target:** Customers reaching Month 3+

**Structure:**
```
Points System:
  - 1 point per $1 spent
  - 100 points = $10 off
  - Bonus: 2x points on birthday month
  
Tier System:
  - Bronze: 0-2 purchases
  - Silver: 3-5 purchases (5% discount)
  - Gold: 6+ purchases (10% discount + free shipping)
```

**Goal:** Increase Month 6 retention by 40%

---

#### 4️⃣ Channel Optimization
**Analysis:** Compare acquisition channel retention

**Method:**
- Merge cohort data with acquisition source (if available)
- Calculate Month 6 retention by channel
- Reallocate budget to highest-retention channels

**Expected Impact:**
- 20-30% improvement in cohort quality
- Lower customer acquisition cost per loyal customer

---

### 📈 Success Metrics

| Metric | Current Baseline | 3-Month Target | 6-Month Target |
|--------|------------------|----------------|----------------|
| Month 1 Retention | [From Cell 17] | +5 percentage pts | +10 percentage pts |
| Month 6 Retention | [From Cell 17] | +3 percentage pts | +7 percentage pts |
| Churn Cliff Drop | [From Cell 17] | -20% reduction | -40% reduction |
| Win-Back Rate | N/A (new) | 10% | 15% |

---

### ⚠️ Red Flags to Monitor

**Declining Retention in New Cohorts:**
- If recent cohorts show worse Month 1 retention than older ones
- Indicates worsening onboarding or product quality
- **Action:** Immediate customer experience audit

**Increasing Churn Cliff:**
- If the churn cliff month is getting steeper
- Customers giving up faster
- **Action:** Survey churned customers, identify pain points

**Wide Cohort Variance:**
- If retention varies wildly between cohorts (30%+ difference)
- Indicates inconsistent customer experience
- **Action:** Standardize onboarding, test for quality issues

---

### 🎯 Implementation Timeline

**Week 1-2:**
- Set up automated email sequences (Day 3, 7, 21)
- Create email templates and automation rules

**Week 3-4:**
- Launch churn cliff intervention campaign
- A/B test win-back offers

**Month 2-3:**
- Design loyalty program structure
- Get stakeholder buy-in, estimate costs

**Month 4+:**
- Launch loyalty program beta
- Monitor retention metrics weekly
- Iterate based on performance

---

### 📊 Reporting Cadence

**Weekly:**
- Track email automation performance (open rates, conversions)
- Monitor win-back campaign results

**Monthly:**
- Update cohort heatmap with new data
- Review Month 1 retention trend
- Adjust tactics based on performance

**Quarterly:**
- Full cohort analysis review
- Compare current cohorts to historical benchmarks
- Present findings to leadership with ROI data

---

**📊 Continue to Section 8 for New vs. Returning Customer Analysis →**

---
```

**🔍 Explanation of Findings:**

**Critical Metrics:**
- **Month 1 retention** = onboarding quality indicator
- **Month 6 retention** = long-term loyalty health check
- **Churn cliff** = pinpoints exact intervention timing

**Actionable Framework:**
- Automated email sequences with specific day triggers
- Loyalty program structure with clear tiers
- Win-back campaigns targeted at churn cliff month

**Implementation Roadmap:**
- Week-by-week plan for first month
- Phased approach for long-term initiatives
- Clear ownership and budget requirements

**Risk Management:**
- Red flags to watch for (declining new cohorts)
- Early warning system for quality issues
- Quarterly review cadence for strategic adjustments

---

## 🎯 SECTION 7 COMPLETION CHECKLIST

- [x] Cell 15: Cohort methodology and reading guide
- [x] Cell 16: Cohort calculation with retention matrix
- [x] Cell 17: Heatmap visualization + retention curve + best/worst cohorts
- [x] Cell 18: Strategic findings with automated campaign plan

**Expected Output:**
- Cohort retention heatmap with % values
- Average retention curve with benchmarks
- Best/worst performing cohorts identified
- Churn cliff month pinpointed
- 4-phase implementation plan (Week 1 → Month 4+)
- Email automation sequences defined
- Loyalty program structure outlined

---

**📂 File Location:** `Section_7_Cohort_Retention_Analysis.md`  
**🔗 Next:** Section 8 - New vs. Returning Customer Analysis
