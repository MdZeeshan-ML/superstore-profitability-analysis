# 📊 SECTION 1: EXECUTIVE DASHBOARD

## Purpose
Provide a 10-second snapshot of business health with key performance indicators (KPIs) and trends.

---

## ⚠️ IMPORTANT: Execution Order

**DO NOT RUN THIS CELL UNTIL YOU'VE COMPLETED SECTION 3 (Data Loading)**

This dashboard requires the `df` DataFrame to be loaded first. Proper execution order:

```
✅ Section 0: Setup → Load libraries
✅ Section 3: Data Loading → Load CSV into 'df'
✅ Section 1: Executive Dashboard → Run this cell
```

---

## 📋 MARKDOWN CELL: Dashboard Introduction

```markdown
---

# 📊 EXECUTIVE DASHBOARD
## Business Health Snapshot

**⚠️ PREREQUISITE: Run Section 3 (Data Loading) first!**

This dashboard provides an at-a-glance view of the business's key metrics:
- **Revenue & Profitability:** Total sales and margin performance
- **Order Metrics:** Transaction volume and average order value
- **Trends:** Monthly revenue patterns
- **Product Performance:** Top profit generators and loss-makers

**Target Audience:** Executives, stakeholders, decision-makers
**Update Frequency:** Re-run this cell whenever data is refreshed

---
```

---

## 💻 CODE CELL: Executive Dashboard Visualization

```python
# Executive Dashboard - REQUIRES data from Section 3

# === DATA VALIDATION CHECK ===
try:
    # Check if df exists and has required columns
    required_columns = ['Sales', 'Profit', 'Order ID', 'Order_Date', 'Sub-Category', 'Category']
    missing_cols = [col for col in required_columns if col not in df.columns]
    
    if missing_cols:
        raise ValueError(f"Missing required columns: {missing_cols}")
    
    if len(df) == 0:
        raise ValueError("DataFrame is empty. Load data first (Section 3).")
    
    print("✅ Data validation passed. Generating dashboard...\n")
    
except NameError:
    print("❌ ERROR: DataFrame 'df' not found!")
    print("📋 SOLUTION: Go to Section 3 and run the data loading cells first.")
    print("   Then come back here and re-run this cell.\n")
    raise NameError("Please load data in Section 3 before running this dashboard.")

# === DASHBOARD GENERATION ===
import matplotlib.patches as mpatches

fig = plt.figure(figsize=(16, 10), facecolor='#f8f9fa')
gs = fig.add_gridspec(3, 4, hspace=0.35, wspace=0.3)

def create_kpi_card(ax, value, label, color, icon):
    """Create a visually appealing KPI card"""
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

# KPI Card 1: Total Revenue
ax1 = fig.add_subplot(gs[0, 0])
total_revenue = df['Sales'].sum()
create_kpi_card(ax1, f"${total_revenue/1e6:.1f}M", 
                "Total Revenue", "#2ecc71", "💰")

# KPI Card 2: Profit Margin
ax2 = fig.add_subplot(gs[0, 1])
margin = (df['Profit'].sum() / df['Sales'].sum() * 100)
create_kpi_card(ax2, f"{margin:.1f}%", 
                "Profit Margin", "#3498db", "📈")

# KPI Card 3: Total Orders
ax3 = fig.add_subplot(gs[0, 2])
total_orders = df['Order ID'].nunique()
create_kpi_card(ax3, f"{total_orders:,}", 
                "Total Orders", "#9b59b6", "🛍")

# KPI Card 4: Average Order Value
ax4 = fig.add_subplot(gs[0, 3])
aov = df['Sales'].sum() / df['Order ID'].nunique()
create_kpi_card(ax4, f"${aov:.0f}", 
                "Avg Order Value", "#e74c3c", "🎯")

# Chart 1: Monthly Revenue Trend
ax5 = fig.add_subplot(gs[1, :2])
monthly = df.groupby(df['Order_Date'].dt.to_period('M'))['Sales'].sum()
ax5.fill_between(range(len(monthly)), monthly.values, alpha=0.3, color='#3498db')
ax5.plot(monthly.values, linewidth=3, color='#3498db', marker='o')
ax5.set_title('📊 Monthly Revenue Trend', fontsize=14, fontweight='bold', pad=15)
ax5.set_xlabel('Month')
ax5.set_ylabel('Revenue ($)')
ax5.grid(alpha=0.3)
ax5.set_xticks(range(0, len(monthly), max(1, len(monthly)//12)))
ax5.set_xticklabels([str(monthly.index[i]) for i in range(0, len(monthly), max(1, len(monthly)//12))], rotation=45)

# Chart 2: Top Profitable Categories
ax6 = fig.add_subplot(gs[1, 2:])
top_cats = df.groupby('Sub-Category')['Profit'].sum().nlargest(7)
ax6.barh(range(len(top_cats)), top_cats.values, color='#2ecc71')
ax6.set_yticks(range(len(top_cats)))
ax6.set_yticklabels(top_cats.index)
ax6.set_xlabel('Profit ($)')
ax6.set_title('🏆 Top Profitable Categories', fontsize=14, fontweight='bold', pad=15)
ax6.grid(alpha=0.3, axis='x')

# Chart 3: Loss-Making Categories
ax7 = fig.add_subplot(gs[2, :2])
worst_cats = df.groupby('Sub-Category')['Profit'].sum().nsmallest(5)
ax7.barh(range(len(worst_cats)), worst_cats.values, color='#e74c3c')
ax7.set_yticks(range(len(worst_cats)))
ax7.set_yticklabels(worst_cats.index)
ax7.set_xlabel('Profit ($)')
ax7.set_title('🚨 Loss-Making Categories', fontsize=14, fontweight='bold', pad=15)
ax7.grid(alpha=0.3, axis='x')

# Chart 4: Category Distribution by Sales
ax8 = fig.add_subplot(gs[2, 2:])
cat_sales = df.groupby('Category')['Sales'].sum().sort_values(ascending=False)
colors_pie = ['#3498db', '#2ecc71', '#e74c3c']
ax8.pie(cat_sales.values, labels=cat_sales.index, autopct='%1.1f%%', 
        colors=colors_pie, startangle=90)
ax8.set_title('📦 Sales by Category', fontsize=14, fontweight='bold', pad=15)

plt.suptitle('🏢 EXECUTIVE DASHBOARD', fontsize=20, fontweight='bold', y=0.98)
plt.tight_layout()
plt.show()

print("\n✅ Dashboard generated successfully")
print("="*60)
print(f"📊 Total Revenue: ${total_revenue:,.2f}")
print(f"💰 Total Profit: ${df['Profit'].sum():,.2f}")
print(f"📈 Profit Margin: {margin:.2f}%")
print(f"🛍 Total Orders: {total_orders:,}")
print(f"🎯 Average Order Value: ${aov:.2f}")
print("="*60)
```

---

## 📊 EXPLANATION: What This Dashboard Shows

```markdown
## 🔍 Dashboard Components Explained

### KPI Cards (Top Row)
1. **Total Revenue ($M):** Aggregate sales across all transactions
   - Green indicates positive performance
   - Use this to track overall business size

2. **Profit Margin (%):** Net profitability ratio
   - Blue color for neutral/analytical metric
   - Industry benchmark: 10-20% is healthy for retail
   - Formula: (Total Profit / Total Sales) × 100

3. **Total Orders:** Number of unique transactions
   - Purple for order-related metrics
   - Higher volume = more customer touchpoints

4. **Average Order Value ($):** Revenue per transaction
   - Red for attention-grabbing metric
   - Formula: Total Sales / Total Orders
   - Key for pricing and bundling strategies

### Visualization Insights

**Monthly Revenue Trend (Middle Left):**
- Identifies seasonal patterns and growth trends
- Area chart shows volume magnitude
- Look for: Peaks (holiday seasons), troughs (slow months), overall direction

**Top Profitable Categories (Middle Right):**
- Horizontal bar chart for easy comparison
- Green = positive contribution
- Focus investment on these high-performers

**Loss-Making Categories (Bottom Left):**
- Red = warning signal
- These require immediate action: price increase, cost reduction, or discontinuation
- Negative values mean these products lose money

**Sales Distribution Pie Chart (Bottom Right):**
- Shows relative size of each category
- Helps identify concentration risk (if one category dominates)
- Useful for portfolio diversification planning

---
```
## ⚙️ Customization Options

**Change Colors:**
```python
# Modify color palette in create_kpi_card function
color_scheme = {
    'revenue': '#2ecc71',    # Green
    'margin': '#3498db',     # Blue
    'orders': '#9b59b6',     # Purple
    'aov': '#e74c3c'         # Red
}
```

**Adjust Time Period:**
```python
# Filter to specific year before generating dashboard
df_2017 = df[df['OrderY'] == 2017]
# Then run dashboard code with df_2017 instead of df
```

**Add More KPIs:**
```python
# Example: Customer Count
ax5 = fig.add_subplot(gs[0, 4])  # Add 5th column
total_customers = df['Customer ID'].nunique()
create_kpi_card(ax5, f"{total_customers:,}", 
                "Total Customers", "#f39c12", "👥")
```

---

## 🚨 Troubleshooting

### Error: NameError: name 'df' is not defined
**Cause:** Data not loaded yet  
**Solution:** Go to Section 3 and run data loading cells first

### Error: KeyError: 'Sales' or 'Profit'
**Cause:** Column names don't match expected format  
**Solution:** Check your CSV column names. Adjust code if using different names:
```python
# If your CSV uses 'Revenue' instead of 'Sales'
df.rename(columns={'Revenue': 'Sales'}, inplace=True)
```

### Warning: Empty DataFrame
**Cause:** Data loaded but contains no rows  
**Solution:** Verify CSV file path and content in Section 3

### Chart looks cluttered
**Cause:** Too many data points  
**Solution:** Filter to recent time period:
```python
# Filter to last 12 months
recent_date = df['Order_Date'].max() - pd.DateOffset(months=12)
df_recent = df[df['Order_Date'] >= recent_date]
# Use df_recent for dashboard
```

---

## 📌 Best Practices

1. **Run After Data Loading:** This dashboard requires the `df` DataFrame from Section 3
2. **Update Regularly:** Re-run when data is refreshed
3. **Export for Presentations:** Save figure using:
   ```python
   plt.savefig('executive_dashboard.png', dpi=300, bbox_inches='tight')
   ```
4. **Mobile Compatibility:** Current size (16x10) works well for desktop; reduce to (12x8) for mobile
5. **Performance:** For large datasets (>100K rows), aggregate before plotting

---

## 🎯 Business Questions This Answers

✅ **How is the business performing overall?** → KPI cards provide instant answer  
✅ **Are we growing or declining?** → Monthly trend line shows trajectory  
✅ **Which products should we promote?** → Top profitable categories  
✅ **What's draining profit?** → Loss-making categories (immediate action needed)  
✅ **Is our portfolio balanced?** → Pie chart shows concentration risk  

---

## 🔗 Recommended Workflow

**First-Time Setup:**
```
1. Section 0 → Load libraries
2. Section 3 → Load data into 'df'
3. Section 1 → Generate this dashboard (YOU ARE HERE)
4. Section 2 → Review key findings
```

**After Completing Full Analysis:**
- Return here and re-run to see updated insights
- Use this as your "homepage" for the analysis
- Share this visual with stakeholders first

**Before Presentations:**
- Run this dashboard last to ensure all data is current
- Export as PNG for slides
- Print KPI summary for handouts

---

## 🔗 Next Steps

After reviewing this dashboard:
1. Proceed to **Section 2: Key Findings** for actionable insights
2. Deep-dive into **Section 4: Discount Analysis** to understand margin drivers
3. Explore **Section 5: RFM Segmentation** to identify high-value customers

```

---

## 📝 Notes for Implementation

- **Dependencies:** Requires matplotlib, seaborn, pandas, numpy (installed in Section 0)
- **Data Required:** `df` DataFrame with columns: `Sales`, `Profit`, `Order ID`, `Order_Date`, `Sub-Category`, `Category`
- **Execution Time:** ~2-3 seconds for typical dataset (10K rows)
- **Output:** Interactive matplotlib figure + printed summary statistics
- **Validation:** Includes try-except block to catch data loading errors

---

## ✅ CHECKLIST: Before Running This Cell

- [ ] Libraries loaded in Section 0
- [ ] Data loaded in Section 3
- [ ] DataFrame named `df` exists
- [ ] Required columns present: Sales, Profit, Order ID, Order_Date, Sub-Category, Category
- [ ] Order_Date converted to datetime format
- [ ] At least 100 rows of data (for meaningful visualizations)

If all checked, proceed with running the dashboard code!

---

**Status:** ✅ Complete (with data validation)  
**Priority:** ⭐⭐⭐ Critical  
**Execution Order:** After Section 3  
**Author:** Mohammad Zeeshan Hussain  
**Last Updated:** January 16, 2026
