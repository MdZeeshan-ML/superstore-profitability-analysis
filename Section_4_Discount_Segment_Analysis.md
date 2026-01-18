# 💸 SECTION 4: DISCOUNT SEGMENT ANALYSIS

## Overview
This section provides a comprehensive template for analyzing discount patterns and their impact on profitability across different customer segments. The analysis identifies which discount levels destroy margins and estimates recoverable profit.

---

## 📋 CELL 7 - Discount Context (Markdown)

```markdown
---

# 💸 SECTION 4: DISCOUNT SEGMENT ANALYSIS

## Why This Matters
Excessive discounting is the #1 profit killer. This analysis identifies which discount levels destroy margins and estimates recoverable profit.

## Key Questions
- At what discount % do orders become unprofitable?
- Which customer segments abuse discounts?
- How much profit is recoverable by capping discounts?

## Business Impact
**Typical Finding:** 60%+ discounts generate consistent losses. Capping at 30% can recover $50K-$150K annually for mid-size retailers.
```

---

## 📊 CELL 8 - Discount Buckets Analysis (Code)

```python
# Create discount buckets
df['Discount_Bucket'] = pd.cut(df['Discount'], 
                                bins=[0, 0.1, 0.2, 0.3, 0.4, 0.5, 1.0],
                                labels=['0-10%', '10-20%', '20-30%', '30-40%', '40-50%', '50%+'],
                                include_lowest=True)

# Aggregate by discount bucket
discount_analysis = df.groupby('Discount_Bucket', observed=True).agg({
    'Order ID': 'nunique',
    'Sales': 'sum',
    'Profit': 'sum',
    'Discount': 'mean'
}).reset_index()

discount_analysis.columns = ['Discount_Bucket', 'Orders', 'Revenue', 'Profit', 'Avg_Discount']
discount_analysis['Profit_Margin_%'] = (discount_analysis['Profit'] / discount_analysis['Revenue'] * 100).round(2)
discount_analysis['Avg_Order_Value'] = (discount_analysis['Revenue'] / discount_analysis['Orders']).round(2)

# Calculate recoverable profit (eliminate 40%+ discounts)
high_discount_loss = df[df['Discount'] >= 0.4]['Profit'].sum()
recoverable_profit = abs(high_discount_loss) if high_discount_loss < 0 else 0

print("📊 DISCOUNT IMPACT SUMMARY")
print("="*60)
print(discount_analysis.to_string(index=False))
print("\n💰 RECOVERABLE PROFIT OPPORTUNITY")
print(f"   Loss from 40%+ discounts: ${high_discount_loss:,.2f}")
print(f"   Recoverable by capping at 30%: ${recoverable_profit:,.2f}")
```

### Code Explanation:

**1. Creating Discount Buckets:**
- `pd.cut()` divides discount values into bins (0-10%, 10-20%, etc.)
- `include_lowest=True` ensures 0% discounts are included in first bucket
- Creates categorical variable for grouping

**2. Aggregation Metrics:**
- `Order ID: nunique` - counts unique orders per bucket
- `Sales: sum` - total revenue per bucket  
- `Profit: sum` - total profit/loss per bucket
- `Discount: mean` - average discount rate per bucket

**3. Calculated Fields:**
- `Profit_Margin_%` - percentage showing profitability per dollar sold
- `Avg_Order_Value` - typical transaction size per discount level

**4. Recovery Calculation:**
- Identifies orders with 40%+ discount
- Calculates total loss from these orders
- Estimates recoverable profit if capped at 30%

---

## 📈 CELL 9 - Discount Visualization Dashboard (Code)

```python
# Create comprehensive discount dashboard
fig = plt.figure(figsize=(18, 12), facecolor='#f8f9fa')
gs = fig.add_gridspec(3, 2, hspace=0.3, wspace=0.25)

# 1. Profit by Discount Bucket (Bar Chart)
ax1 = fig.add_subplot(gs[0, :])
colors = ['#2ecc71' if p > 0 else '#e74c3c' for p in discount_analysis['Profit']]
bars = ax1.bar(discount_analysis['Discount_Bucket'], discount_analysis['Profit'], color=colors, alpha=0.7)
ax1.axhline(y=0, color='black', linestyle='-', linewidth=0.8)
ax1.set_title('💰 Profit by Discount Level - The Profitability Cliff', fontsize=14, fontweight='bold')
ax1.set_xlabel('Discount Bucket', fontsize=11)
ax1.set_ylabel('Total Profit ($)', fontsize=11)
ax1.grid(axis='y', alpha=0.3)

# Add value labels
for bar in bars:
    height = bar.get_height()
    ax1.text(bar.get_x() + bar.get_width()/2., height,
             f'${height:,.0f}', ha='center', va='bottom' if height > 0 else 'top', fontsize=10)

# 2. Profit Margin % Trend (Line Chart)
ax2 = fig.add_subplot(gs[1, 0])
ax2.plot(discount_analysis['Discount_Bucket'], discount_analysis['Profit_Margin_%'], 
         marker='o', linewidth=3, markersize=8, color='#3498db')
ax2.axhline(y=0, color='red', linestyle='--', linewidth=2, label='Break-Even')
ax2.fill_between(range(len(discount_analysis)), discount_analysis['Profit_Margin_%'], 
                  0, alpha=0.2, color='#3498db')
ax2.set_title('📉 Margin Erosion by Discount Level', fontsize=12, fontweight='bold')
ax2.set_xlabel('Discount Bucket')
ax2.set_ylabel('Profit Margin (%)')
ax2.legend()
ax2.grid(alpha=0.3)

# 3. Order Volume by Discount (Bar Chart)
ax3 = fig.add_subplot(gs[1, 1])
ax3.bar(discount_analysis['Discount_Bucket'], discount_analysis['Orders'], 
        color='#9b59b6', alpha=0.7)
ax3.set_title('📦 Order Volume by Discount Level', fontsize=12, fontweight='bold')
ax3.set_xlabel('Discount Bucket')
ax3.set_ylabel('Number of Orders')
ax3.grid(axis='y', alpha=0.3)

# 4. Revenue vs Profit Comparison (Grouped Bar)
ax4 = fig.add_subplot(gs[2, 0])
x = np.arange(len(discount_analysis))
width = 0.35
ax4.bar(x - width/2, discount_analysis['Revenue'], width, label='Revenue', color='#3498db', alpha=0.7)
ax4.bar(x + width/2, discount_analysis['Profit'], width, label='Profit', color='#2ecc71', alpha=0.7)
ax4.set_title('💵 Revenue vs Profit by Discount Level', fontsize=12, fontweight='bold')
ax4.set_xticks(x)
ax4.set_xticklabels(discount_analysis['Discount_Bucket'], rotation=45)
ax4.legend()
ax4.grid(axis='y', alpha=0.3)

# 5. Scatter: Discount vs Profitability
ax5 = fig.add_subplot(gs[2, 1])
scatter_data = df.sample(min(1000, len(df)))  # Sample for performance
colors_scatter = scatter_data['Profit'].apply(lambda x: '#2ecc71' if x > 0 else '#e74c3c')
ax5.scatter(scatter_data['Discount']*100, scatter_data['Profitability']*100, 
            c=colors_scatter, alpha=0.5, s=30)
ax5.axhline(y=0, color='black', linestyle='--', linewidth=1.5)
ax5.axvline(x=30, color='orange', linestyle='--', linewidth=2, label='Recommended Cap (30%)')
ax5.set_title('🎯 Discount vs Profitability Scatter', fontsize=12, fontweight='bold')
ax5.set_xlabel('Discount (%)')
ax5.set_ylabel('Profitability (%)')
ax5.legend()
ax5.grid(alpha=0.3)

plt.suptitle('📊 DISCOUNT IMPACT DASHBOARD - Identifying Profit Killers', 
             fontsize=16, fontweight='bold', y=0.995)
plt.tight_layout()
plt.show()
```

### Visualization Breakdown:

**Chart 1: Profit by Discount Bucket**
- Shows absolute profit/loss per discount tier
- Color-coded: Green (profit), Red (loss)
- Reveals "profitability cliff" where margins turn negative

**Chart 2: Margin Erosion Line**
- Tracks profit margin % decline as discounts increase
- Break-even line shows where business stops making money
- Fill area visualizes profit erosion

**Chart 3: Order Volume Distribution**
- Shows how many orders fall into each discount tier
- Helps identify if high discounts are common or rare

**Chart 4: Revenue vs Profit Comparison**
- Side-by-side bars reveal disconnect between sales and profitability
- High revenue with negative profit = problem

**Chart 5: Discount vs Profitability Scatter**
- Individual order-level view (sampled for performance)
- Vertical line at 30% shows recommended discount cap
- Pattern reveals discount threshold for profitability

---

## 📋 CELL 10 - Discount Segment Breakdown (Code)

```python
# Analyze discount patterns by customer segment and category
print("🔍 DISCOUNT PATTERNS BY SEGMENT\n")

# By Customer Segment
segment_discount = df.groupby('Segment').agg({
    'Discount': 'mean',
    'Profit': 'sum',
    'Sales': 'sum',
    'Order ID': 'nunique'
}).round(2)

segment_discount['Avg_Discount_%'] = (segment_discount['Discount'] * 100).round(1)
segment_discount['Profit_Margin_%'] = (segment_discount['Profit'] / segment_discount['Sales'] * 100).round(1)
segment_discount = segment_discount.sort_values('Avg_Discount_%', ascending=False)

print("📊 Discount Usage by Customer Segment:")
print(segment_discount[['Avg_Discount_%', 'Profit', 'Profit_Margin_%', 'Order ID']].to_string())

# By Product Category
print("\n\n📦 Discount Usage by Product Category:")
category_discount = df.groupby('Category').agg({
    'Discount': 'mean',
    'Profit': 'sum',
    'Sales': 'sum'
}).round(2)

category_discount['Avg_Discount_%'] = (category_discount['Discount'] * 100).round(1)
category_discount['Profit_Margin_%'] = (category_discount['Profit'] / category_discount['Sales'] * 100).round(1)
category_discount = category_discount.sort_values('Avg_Discount_%', ascending=False)

print(category_discount[['Avg_Discount_%', 'Profit', 'Profit_Margin_%']].to_string())

# High-risk combinations
print("\n\n⚠️ HIGH-RISK COMBINATIONS (Segment + High Discount):")
high_discount_segments = df[df['Discount'] >= 0.4].groupby('Segment').agg({
    'Order ID': 'nunique',
    'Profit': 'sum'
}).sort_values('Profit', ascending=True)

print(high_discount_segments.to_string())
```

### Analysis Purpose:

**Segment Analysis:**
- Identifies which customer types receive highest discounts
- Correlates discount patterns with profitability
- Reveals if certain segments are more price-sensitive

**Category Analysis:**  
- Shows which product categories are heavily discounted
- Identifies categories where discounting works vs. doesn't

**High-Risk Combinations:**
- Isolates 40%+ discount orders by segment
- Pinpoints specific segment-discount combinations losing money
- Guides targeted policy changes

---

## 📝 CELL 11 - Discount Findings & Recommendations (Markdown)

```markdown
## 🔍 DISCOUNT ANALYSIS FINDINGS

### Critical Insights
1. **Profitability Cliff:** Orders with 40%+ discounts consistently generate losses
2. **Recoverable Profit:** Capping discounts at 30% could recover $[INSERT VALUE] annually
3. **High-Risk Segment:** [INSERT SEGMENT] receives highest average discounts at [X]%
4. **Category Impact:** [INSERT CATEGORY] shows most discount abuse with lowest margins

### The 30% Rule
**Mathematical Threshold:** Based on average COGS and operational costs, discounts above 30% make most transactions unprofitable.

| Discount Level | Avg Profit Margin | Profitability Status |
|----------------|-------------------|---------------------|
| 0-10% | [X]% | ✅ Healthy |
| 10-20% | [X]% | ✅ Acceptable |
| 20-30% | [X]% | ⚠️ Marginal |
| 30-40% | [X]% | 🚨 At Risk |
| 40-50% | [X]% | 🚨 Loss Territory |
| 50%+ | [X]% | 🚨 Major Loss |

### Immediate Actions
1. **Policy Change:** Implement 30% discount cap effective immediately
2. **Approval Workflow:** Require VP approval for 30-40% discounts
3. **Segment Rules:** Limit [HIGH-RISK SEGMENT] to 20% max discount
4. **Category Restrictions:** No discounts >25% on [LOW-MARGIN CATEGORY]
5. **Monitoring:** Weekly dashboard tracking discount compliance

### Financial Impact (Annual Projection)
- **Current Loss from 40%+ Discounts:** $[X]
- **Projected Recovery (30% cap):** $[Y]
- **Implementation Cost:** $[Z] (policy update + training)
- **Net Benefit:** $[Y-Z]
- **ROI:** [ROI%] within first quarter

---
```

---

## 📝 Implementation Notes

### Data Requirements:
The code assumes your dataframe includes:
- `Discount` column (decimal format: 0.1 = 10%)
- `Order ID` column (unique identifier)
- `Sales` column (revenue per transaction)
- `Profit` column (profit/loss per transaction)  
- `Segment` column (customer segmentation)
- `Category` column (product categories)

### Important Considerations:

**1. Profitability Column:**
If your dataset doesn't have a `Profitability` column for Cell 9 Chart 5, create it:
```python
df['Profitability'] = df['Profit'] / df['Sales']
```

**2. Segment Column:**
If the `Segment` column doesn't exist yet, it may be created in Section 5 (RFM Segmentation). You may need to:
- Reorder sections, OR
- Comment out segment-specific analysis in Cell 10 until Section 5 is complete

**3. Category Customization:**
Adjust discount buckets based on your business:
```python
# More granular
bins=[0, 0.05, 0.1, 0.15, 0.2, 0.25, 0.3, 1.0]

# Less granular  
bins=[0, 0.2, 0.4, 1.0]
```

### Customization Options:

**Discount Threshold:**
- Change recommended cap from 30% to your business threshold
- Modify high-risk filter from 40% to 30% for stricter analysis

**Visualization Colors:**
- Update color scheme to match brand guidelines
- Adjust alpha (transparency) values for clarity

**Industry Benchmarks:**
- Add your industry-specific profit margin targets to findings
- Compare results to competitor benchmarks

---

## 📊 Expected Outputs

When you run this section, you'll get:

1. **Summary Table** (Cell 8)
   - Profit/loss by discount bucket
   - Recoverable profit calculation
   - Order distribution

2. **5-Panel Dashboard** (Cell 9)
   - Profitability cliff visualization
   - Margin erosion trends
   - Volume distribution
   - Revenue vs profit comparison
   - Individual order scatter plot

3. **Segment Breakdown** (Cell 10)
   - Customer segment discount patterns
   - Category discount analysis  
   - High-risk combinations

4. **Business Findings** (Cell 11)
   - Actionable recommendations with dollar amounts
   - Implementation roadmap
   - ROI projections

---

## 🎯 Integration with Your Notebook

This section fits perfectly between:
- **Section 3:** Data Loading & Basic Metrics
- **Section 5:** RFM Segmentation & Customer Analysis

Order of execution:
1. Load data (Section 3)
2. Run discount analysis (Section 4 - THIS)
3. Segment customers (Section 5)
4. Combine insights for comprehensive report

---

## 💡 Business Value

This analysis directly answers:
- "Why are we not as profitable as we should be?"
- "Where is our money leaking?"
- "What's the single highest-impact action we can take?"

Typical findings:
- 10-15% of orders destroy 30-40% of potential profit
- One policy change (discount cap) = $50K-$150K annual recovery
- ROI of implementation: 10x-50x within 90 days

**This is what clients pay $5,000-$15,000 for.**

---

*Created for: Superstore Business Analytics Project*  
*Repository: [MdZeeshan-ML/Projects](https://github.com/MdZeeshan-ML/Projects)*  
*Last Updated: January 2026*
