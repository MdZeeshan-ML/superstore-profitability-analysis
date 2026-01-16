# 📊 SUPERSTORE PROFITABILITY ANALYSIS - NOTEBOOK STRUCTURE
## Complete Gumroad Product Architecture

---

## 🎯 PRODUCT OVERVIEW

**What Customers Get:**
```
📦 SuperStore_Complete_Bundle.zip
├── 📄 1_Executive_Report.pdf (5-8 pages, mobile-friendly)
├── 📊 2_Interactive_Analysis.html (full notebook with live charts)
└── 📓 3_Source_Notebook.ipynb (editable for customization)
```

**Total Notebook:** 35-40 cells | 8-page PDF | ~5MB HTML | Editable .ipynb

---

## 📋 COMPLETE NOTEBOOK CELL-BY-CELL STRUCTURE

### **CELL 1: Title & Overview** (Markdown)

```markdown
# 🏢 SUPERSTORE PROFITABILITY ANALYSIS
## Revenue Recovery Roadmap - Executive Edition

**WHAT YOU'LL LEARN:**
✅ Recoverable profit from discount optimization  
✅ 4 customer segments driving 80% of revenue  
✅ Geographic expansion opportunities  
✅ Product rationalization savings  

**VIEWING GUIDE:**
📄 Quick insights → Read Executive Summary (15 min)  
📊 Deep dive → Explore interactive charts below  
🔧 Customize → Edit cells and re-run analysis  
```

---

### **SECTIONS OVERVIEW:**

| Section | Topic | Cells | Priority | Status |
|---------|-------|-------|----------|--------|
| 0 | Title & Setup | 2 | ⭐⭐⭐ | Template below |
| 1 | Executive Dashboard | 1 | ⭐⭐⭐ | Template below |
| 2 | Key Findings | 1 | ⭐⭐⭐ | Template below |
| 3 | Data Loading | 2 | ⭐⭐⭐ | Template below |
| 4 | Discount Analysis | 3 | ✅ | You have this |
| 5 | RFM Segmentation | 4 | ⭐⭐⭐ | Template below |
| 6 | Customer Lifetime Value | 4 | ⭐⭐ | Template below |
| 7 | Cohort Retention | 4 | ⭐⭐ | Template below |
| 8 | New vs Returning | 3 | ⭐ | Add later |
| 9 | Geographic Analysis | 3 | ⭐ | Add later |
| 10 | Product Analysis | 3 | ✅ | You have this |
| 11 | Recommendations | 1 | ⭐⭐⭐ | Template below |

**Total:** ~35 cells

---

## 🎨 COMPLETE CELL TEMPLATES

### **SECTION 0: Setup (Cells 1-2)**

**Cell 1 - Title (Markdown):**
```markdown
# 🏢 SUPERSTORE PROFITABILITY ANALYSIS
## Revenue Recovery Roadmap - Executive Edition

**Author:** [Your Name]  
**Date:** January 2026  
**Version:** 1.0

---

## 📊 WHAT YOU'LL DISCOVER

✅ **Discount Optimization:** Identify profit-destroying discount levels  
✅ **Customer Segmentation:** Which customers drive 80% of revenue  
✅ **Lifetime Value:** Calculate long-term customer worth  
✅ **Retention Strategy:** When and why customers churn  
✅ **Geographic Opportunities:** Where to expand next  

---

## 📖 HOW TO USE THIS ANALYSIS

**For Executives:**  
→ Read the Executive Dashboard (next cell) for 10-second overview  
→ Review Key Findings section for action items  
→ Share this PDF with stakeholders  

**For Analysts:**  
→ Run all cells (Kernel → Restart & Run All)  
→ Modify data source in Cell 4  
→ Export custom report when complete  

**For Developers:**  
→ Clone this notebook  
→ Adapt code for your dataset  
→ Integrate into production dashboards  
```

**Cell 2 - Import Libraries (Code):**
```python
# Setup and Configuration
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import plotly.io as pio
from datetime import datetime
import warnings
warnings.filterwarnings('ignore')

# Configure plotting
plt.style.use('seaborn-v0_8-darkgrid')
sns.set_palette("husl")
pio.renderers.default = 'notebook'  # Preserves interactivity

print("✅ Libraries loaded successfully")
```

---

### **SECTION 1: Executive Dashboard (Cell 3)**

**Cell 3 - Dashboard (Code):**
```python
# Executive Dashboard - Run after loading data

import matplotlib.patches as mpatches

fig = plt.figure(figsize=(16, 10), facecolor='#f8f9fa')
gs = fig.add_gridspec(3, 4, hspace=0.35, wspace=0.3)

def create_kpi_card(ax, value, label, color, icon):
    ax.add_patch(mpatches.FancyBboxPatch((0.1, 0.2), 0.8, 0.6, 
                 boxstyle="round,pad=0.1", edgecolor=color, 
                 facecolor=color, alpha=0.15, linewidth=3))
    ax.text(0.5, 0.65, icon, ha='center', va='center', 
            fontsize=40, color=color)
    ax.text(0.5, 0.45, value, ha='center', va='center', 
            fontsize=28, fontweight='bold', color=color)
    ax.text(0.5, 0.25, label, ha='center', va='center', 
            fontsize=12, color='#333')
    ax.set_xlim(0, 1)
    ax.set_ylim(0, 1)
    ax.axis('off')

# KPI Cards
ax1 = fig.add_subplot(gs[0, 0])
create_kpi_card(ax1, f"${df['Sales'].sum()/1e6:.1f}M", 
                "Total Revenue", "#2ecc71", "💰")

ax2 = fig.add_subplot(gs[0, 1])
margin = (df['Profit'].sum() / df['Sales'].sum() * 100)
create_kpi_card(ax2, f"{margin:.1f}%", 
                "Profit Margin", "#3498db", "📈")

ax3 = fig.add_subplot(gs[0, 2])
create_kpi_card(ax3, f"{df['Order_ID'].nunique():,}", 
                "Total Orders", "#9b59b6", "🛍")

ax4 = fig.add_subplot(gs[0, 3])
aov = df['Sales'].sum() / df['Order_ID'].nunique()
create_kpi_card(ax4, f"${aov:.0f}", 
                "Avg Order Value", "#e74c3c", "🎯")

# Monthly trend
ax5 = fig.add_subplot(gs[1, :2])
monthly = df.groupby(df['Order_Date'].dt.to_period('M'))['Sales'].sum()
ax5.fill_between(range(len(monthly)), monthly.values, alpha=0.3, color='#3498db')
ax5.plot(monthly.values, linewidth=3, color='#3498db', marker='o')
ax5.set_title('📊 Monthly Revenue Trend', fontsize=14, fontweight='bold')
ax5.grid(alpha=0.3)

# Top products
ax6 = fig.add_subplot(gs[1, 2:])
top_cats = df.groupby('Sub-Category')['Profit'].sum().nlargest(7)
ax6.barh(range(len(top_cats)), top_cats.values, color='#2ecc71')
ax6.set_yticks(range(len(top_cats)))
ax6.set_yticklabels(top_cats.index)
ax6.set_title('🏆 Top Profitable Categories', fontsize=14, fontweight='bold')

# Loss-makers
ax7 = fig.add_subplot(gs[2, :2])
worst_cats = df.groupby('Sub-Category')['Profit'].sum().nsmallest(5)
ax7.barh(range(len(worst_cats)), worst_cats.values, color='#e74c3c')
ax7.set_yticks(range(len(worst_cats)))
ax7.set_yticklabels(worst_cats.index)
ax7.set_title('🚨 Loss-Making Categories', fontsize=14, fontweight='bold')

plt.suptitle('🏢 EXECUTIVE DASHBOARD', fontsize=20, fontweight='bold', y=0.98)
plt.tight_layout()
plt.show()
```

---

### **SECTION 2: Key Findings (Cell 4)**

**Cell 4 - Findings (Markdown):**
```markdown
# 💡 KEY BUSINESS FINDINGS

## 🎯 30-Day Action Plan

### 1️⃣ Discount Optimization → **Recoverable Profit**
- **Problem:** 60%+ discounts generate losses
- **Action:** Cap discounts at 30%, require approval above
- **Timeline:** Implement by Feb 15

### 2️⃣ Customer Retention → **Value at Risk**
- **Problem:** High-value customers showing churn signals
- **Action:** Launch win-back campaign within 14 days
- **Timeline:** Campaign live by Feb 1

### 3️⃣ Product Rationalization → **Annual Loss Eliminated**
- **Problem:** Specific categories consistently unprofitable
- **Action:** Discontinue bottom SKUs, renegotiate terms
- **Timeline:** Phase-out by Q2

### 4️⃣ Geographic Expansion → **Growth Opportunity**
- **Problem:** Underperforming regions need attention
- **Action:** Reallocate marketing budget to high-potential areas
- **Timeline:** Immediate budget shift

---

## 📊 Supporting Analysis Below
*Detailed visualizations and methodology follow in sections 3-11*
```

---

### **SECTION 3: Data Loading (Cells 5-6)**

**Cell 5 - Load Data (Code):**
```python
# Load and preprocess data
df = pd.read_csv('Sample - Superstore.csv', encoding='ISO-8859-1')

# Date conversion
df['Order_Date'] = pd.to_datetime(df['Order Date'], format='%m/%d/%Y')
df['Ship_Date'] = pd.to_datetime(df['Ship Date'], format='%m/%d/%Y')

# Extract components
df['OrderY'] = df['Order_Date'].dt.year
df['OrderM'] = df['Order_Date'].dt.month
df['OrderD'] = df['Order_Date'].dt.day

# Calculated fields
df['Profitability'] = df['Profit'] / df['Sales']

print(f"✅ Loaded {len(df):,} orders")
print(f"📊 Date Range: {df['Order_Date'].min().date()} to {df['Order_Date'].max().date()}")
print(f"👥 Customers: {df['Customer ID'].nunique():,}")
print(f"💰 Revenue: ${df['Sales'].sum():,.2f}")
```

**Cell 6 - Data Overview (Markdown):**
```markdown
---

## 📋 DATASET OVERVIEW

**Source:** Superstore Sales Data (Kaggle)  
**Records:** 9,994 transactions  
**Period:** 2014-2017 (4 years)  
**Scope:** United States retail

### Key Columns
- Order_Date: Transaction timestamp
- Customer_ID: Unique identifier
- Sales: Revenue ($)
- Profit: Margin ($)
- Discount: Promotional level (0-1)
- Category/Sub-Category: Product classification
```

---

### **SECTION 5: RFM Segmentation (Cells 7-10)**

**Cell 7 - RFM Context (Markdown):**
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

**Cell 8 - RFM Calculation (Code):**
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

**Cell 9 - Pareto Chart (Code):**
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

**Cell 10 - RFM Findings (Markdown):**
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

### **SECTION 6: Customer Lifetime Value (Cells 11-14)**

**Cell 11 - CLV Context (Markdown):**
```markdown
# 💵 SECTION 6: CUSTOMER LIFETIME VALUE (CLV)

## Why This Matters
CLV tells you how much to invest in acquiring and retaining each customer.

## Calculation Method
```
CLV = (Average Order Value) × (Purchase Frequency) × (Customer Lifespan)
```

## Business Impact
- Justify customer acquisition costs
- Prioritize high-value segments
- Set retention budgets
```

**Cell 12 - CLV Calculation (Code):**
```python
# Calculate CLV components
customer_stats = df.groupby('Customer ID').agg({
    'Order ID': 'nunique',  # Frequency
    'Sales': 'sum',         # Total revenue
    'Order_Date': ['min', 'max']  # Lifespan
}).reset_index()

customer_stats.columns = ['Customer_ID', 'Frequency', 'Total_Sales', 'First_Order', 'Last_Order']

# Calculate lifespan in days
customer_stats['Lifespan_Days'] = (customer_stats['Last_Order'] - customer_stats['First_Order']).dt.days
customer_stats['Lifespan_Days'] = customer_stats['Lifespan_Days'].replace(0, 1)  # Avoid division by zero

# Calculate metrics
customer_stats['AOV'] = customer_stats['Total_Sales'] / customer_stats['Frequency']
customer_stats['Purchase_Rate'] = customer_stats['Frequency'] / customer_stats['Lifespan_Days'] * 365

# Simple CLV (3-year projection)
customer_stats['CLV_3Y'] = customer_stats['AOV'] * customer_stats['Purchase_Rate'] * 3

print(f"💰 Average CLV (3-year): ${customer_stats['CLV_3Y'].mean():,.2f}")
print(f"🏆 Top 10% CLV: ${customer_stats['CLV_3Y'].quantile(0.9):,.2f}")
```

**Cell 13 - CLV Distribution (Code):**
```python
# Visualize CLV distribution
fig = px.histogram(customer_stats, x='CLV_3Y', nbins=50,
                   title='📊 Customer Lifetime Value Distribution (3-Year)',
                   labels={'CLV_3Y': 'CLV ($)', 'count': 'Customers'})

fig.add_vline(x=customer_stats['CLV_3Y'].median(), line_dash="dash", 
              line_color="red", annotation_text="Median")
fig.show()

# Top customers by CLV
top_clv = customer_stats.nlargest(10, 'CLV_3Y')[['Customer_ID', 'CLV_3Y', 'AOV', 'Frequency']]
print("⭐ Top 10 Customers by CLV:")
print(top_clv)
```

**Cell 14 - CLV Findings (Markdown):**
```markdown
## 🔍 CLV FINDINGS

### Key Metrics
- **Average 3-year CLV:** [Insert calculated value]
- **Top 10% threshold:** [Insert value]
- **Acquisition cost ceiling:** Max 30% of CLV

### Strategic Actions
1. **VIP Program:** Target customers with CLV > 90th percentile
2. **Acquisition Budget:** Cap CAC at $[CLV × 0.3]
3. **Retention Priority:** Focus on customers at CLV median+

---
```

---

### **SECTION 7: Cohort Retention (Cells 15-18)**

**Cell 15 - Cohort Context (Markdown):**
```markdown
# 📆 SECTION 7: COHORT RETENTION ANALYSIS

## Why This Matters
Cohort analysis reveals WHEN customers churn and which acquisition periods performed best.

## What We're Tracking
- **Cohort:** Customers grouped by first purchase month
- **Retention Rate:** % returning in subsequent months
- **Churn Patterns:** When do customers stop buying?
```

**Cell 16 - Cohort Preparation (Code):**
```python
# Create cohort table
df['OrderMonth'] = df['Order_Date'].dt.to_period('M')

# Get first purchase month for each customer
cohort_data = df.groupby('Customer ID')['OrderMonth'].min().reset_index()
cohort_data.columns = ['Customer_ID', 'CohortMonth']

# Merge back
df_cohort = df.merge(cohort_data, left_on='Customer ID', right_on='Customer_ID')

# Calculate periods since cohort
df_cohort['CohortIndex'] = (df_cohort['OrderMonth'] - df_cohort['CohortMonth']).apply(lambda x: x.n)

# Create cohort pivot
cohort_counts = df_cohort.groupby(['CohortMonth', 'CohortIndex'])['Customer_ID'].nunique().reset_index()
cohort_pivot = cohort_counts.pivot(index='CohortMonth', columns='CohortIndex', values='Customer_ID')

# Calculate retention rates
cohort_size = cohort_pivot.iloc[:, 0]
retention = cohort_pivot.divide(cohort_size, axis=0) * 100

print("✅ Cohort analysis prepared")
print(f"📊 Cohorts tracked: {len(cohort_pivot)} months")
```

**Cell 17 - Retention Heatmap (Code):**
```python
# Create retention heatmap
plt.figure(figsize=(16, 10))
sns.heatmap(retention, annot=True, fmt='.0f', cmap='RdYlGn', 
            vmin=0, vmax=100, linewidths=0.5)

plt.title('🔥 Cohort Retention Heatmap (%)', fontsize=16, fontweight='bold')
plt.xlabel('Months Since First Purchase')
plt.ylabel('Cohort (First Purchase Month)')
plt.tight_layout()
plt.show()

# Calculate average retention by month
avg_retention = retention.mean(axis=0)
print("📊 Average Retention by Month:")
print(avg_retention.head(12))
```

**Cell 18 - Cohort Findings (Markdown):**
```markdown
## 🔍 COHORT FINDINGS

### Retention Patterns
- **Month 1 retention:** [Insert %] - Critical baseline
- **Month 6 retention:** [Insert %] - Long-term viability
- **Churn cliff:** Highest drop occurs at month [X]

### Action Plan
1. **30-day onboarding:** Maximize Month 1 retention
2. **90-day win-back:** Target customers at churn cliff
3. **6-month loyalty:** Incentivize continued purchases

---
```

---

### **SECTION 11: Recommendations (Cell 19)**

**Cell 19 - Final Recommendations (Markdown):**
```markdown
# 🎯 EXECUTIVE RECOMMENDATIONS

## 🚀 IMMEDIATE ACTIONS (0-30 days)

### 1. Discount Policy Reform
**Current State:** 60%+ discounts destroying margins  
**Action:** Implement 30% discount cap with VP approval required above  
**Expected Impact:** $[X] annual profit recovery  
**Owner:** Pricing Team  
**Deadline:** February 15, 2026

### 2. At-Risk Customer Campaign
**Current State:** High-value customers showing churn signals  
**Action:** Launch personalized win-back emails with 20% incentive  
**Expected Impact:** Retain [X]% of at-risk segment  
**Owner:** CRM Team  
**Deadline:** February 1, 2026

---

## 📈 SHORT-TERM INITIATIVES (30-90 days)

### 3. VIP Loyalty Program
**Target:** Champions segment (top 15% by CLV)  
**Benefits:** Early access, free shipping, dedicated support  
**Investment:** $[X] setup + $[Y] annual  
**ROI:** [Z]% retention increase = $[A] profit

### 4. Product Line Rationalization
**Action:** Discontinue bottom 10% SKUs by profit  
**Savings:** $[X] annual loss elimination  
**Timeline:** Phase-out by Q2 2026

---

## 🎯 STRATEGIC PROJECTS (90+ days)

### 5. Geographic Expansion
**Priority Markets:** [Regions from analysis]  
**Investment:** Reallocate 40% of underperforming region budgets  
**Expected Growth:** [X]% revenue increase in target markets

### 6. Retention Automation
**Technology:** Implement predictive churn model  
**Trigger:** Automated outreach at 60-day inactivity  
**Impact:** 15-20% churn reduction

---

## 📊 SUCCESS METRICS

| Initiative | Metric | Current | Target | Timeline |
|------------|--------|---------|--------|----------|
| Discount Cap | Avg Discount | [X]% | 30% | Feb 15 |
| At-Risk Campaign | Retention Rate | [Y]% | [Y+10]% | Mar 1 |
| VIP Program | Champions Retention | [Z]% | 95% | Q2 |
| Product Rationalization | Loss from Bottom 10% | $[A] | $0 | Q2 |

---

## 📞 NEXT STEPS

1. **Schedule stakeholder review:** Present findings to executive team
2. **Assign owners:** Delegate initiatives to department heads
3. **Set up tracking:** Weekly dashboard for key metrics
4. **Budget allocation:** Approve investment for Months 1-3

**For questions or custom analysis, contact:** [Your Email]

---

✅ **Analysis Complete** | Version 1.0 | January 2026
```

---

## 📦 EXPORT COMMANDS

### Create All 3 Delivery Files

```bash
# 1. PDF Export
jupyter nbconvert SuperStore_Analysis.ipynb \
    --to webpdf \
    --no-input \
    --output Executive_Report

# 2. HTML Export  
jupyter nbconvert SuperStore_Analysis.ipynb \
    --to html \
    --output Interactive_Analysis

# 3. Create ZIP Bundle
zip -r SuperStore_Complete_Bundle.zip \
    Executive_Report.pdf \
    Interactive_Analysis.html \
    SuperStore_Analysis.ipynb \
    README.txt
```

---

## ✅ PRE-LAUNCH CHECKLIST

**Technical:**
- [ ] All cells run without errors
- [ ] Plotly charts use `pio.renderers.default = 'notebook'`
- [ ] PDF renders correctly on mobile
- [ ] HTML works offline with interactive charts

**Content:**
- [ ] Executive Dashboard on PDF page 1
- [ ] Key Findings readable in 3 minutes
- [ ] Every chart has business context
- [ ] Dollar amounts calculated (not placeholders)

**Delivery:**
- [ ] ZIP contains all 3 files + README
- [ ] README includes viewing instructions
- [ ] Support email added

---

## 🎯 IMPLEMENTATION TIMELINE

**2:30-3:30 PM:** Add RFM segmentation (Cells 7-10)  
**3:30-4:15 PM:** Add CLV calculation  
**4:15-5:00 PM:** Add Cohort retention  
**5:00-5:30 PM:** Test & debug  

**6:30-7:00 PM:** Add New vs Returning  
**7:00-7:45 PM:** Add Geographic heatmap  
**7:45-8:30 PM:** Create Executive Dashboard  
**8:30-9:30 PM:** Export & package  

---

**FINAL PRODUCT:** 35-40 cells → 8-page PDF + 5MB HTML + editable .ipynb

**GUMROAD PRICE:** $49-79 (3-format bundle justifies premium)