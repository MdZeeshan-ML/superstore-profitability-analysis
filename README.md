# 📊 SuperStore Profitability Analysis
**$135,376 Profit Recovery | RFM Segmentation | Customer Lifetime Value Analytics**

<div align="center">

[![Python](https://img.shields.io/badge/Python-5⭐%20HackerRank-gold?logo=python&logoColor=white)](https://www.hackerrank.com/Blanc_coder)
[![Problem Solving](https://img.shields.io/badge/Problem%20Solving-3⭐-success)](https://www.hackerrank.com/Blanc_coder)
[![SQL](https://img.shields.io/badge/SQL-Intermediate-blue?logo=postgresql)](https://www.hackerrank.com/certificates/95324fb8b70b)
[![IIT Madras](https://img.shields.io/badge/IIT%20Madras-Data%20Science-orange)](https://study.iitm.ac.in/)
[![CGPA](https://img.shields.io/badge/CGPA-8.83-brightgreen)](https://study.iitm.ac.in/)

</div>

---

## 👤 About Me

**Mohammad Zeeshan Hussain**  
🎓 Data Science (BS) @ CODE - IIT Madras | CGPA: 8.83  
💻 Python Expert (5⭐ HackerRank) | SQL Certified  
📍 Delhi, India

**Skills:** Python (Pandas, NumPy, Matplotlib, Seaborn, Plotly) | SQL | Data Cleaning | EDA | Statistical Analysis | RFM Segmentation | CLV Modeling

📧 **Contact:** ig.zeeshad@gmail.com  
🐙 **GitHub:** [@MdZeeshan-ML](https://github.com/MdZeeshan-ML)  
💼 **HackerRank:** [@Blanc_coder](https://www.hackerrank.com/Blanc_coder)

---

## 🎯 Executive Summary

Analyzed 9,994 retail transactions (2014-2017) to identify **$135,376 in annual recoverable profit** through:
- Discount policy optimization (30% cap implementation)
- Customer segmentation (9 distinct segments)
- Retention strategy improvements (12.1% → 20% target)

### Business Impact at a Glance

| Metric | Finding | Action |
|--------|---------|--------|
| **Profit at Risk** | $135,376/year from 30%+ discounts | Cap discounts at 29.9% |
| **ROI** | 773% in Year 1 | Implementation cost: $15,500 |
| **High-Risk Loss** | Consumer segment: -$64,216 | Restrict to 20% max discount |
| **Retention Crisis** | 87.9% churn in Month 1 | Launch 30-day activation campaign |
| **VIP Opportunity** | Top 10% = $13,021 avg CLV | Create Platinum tier program |

---

## 📊 Key Visualizations

### Executive Dashboard
![Executive Dashboard](images/dashboard.png)
*8-panel KPI dashboard showing revenue ($2.3M), profit margin (12.5%), order trends, and category performance*

### Discount Impact Analysis
![Discount Analysis](images/discount_impact.png)
*Critical finding: 30%+ discounts create -119% profit margin. Orders with 20-30% discounts lose $10.05 per transaction.*

### Customer Segmentation (Pareto)
![RFM Segments](images/pareto_chart.png)
*5 segments drive 80% of profit: Champions (21.1%), Loyal (18.1%), Potential Loyalists (16.9%), At Risk (12.4%), Hibernating (9.8%)*

---

## 🔍 Key Findings

### 1. Discount Optimization
**Problem:** Discounts >20% consistently generate losses

| Discount Range | Profit Margin | Orders | Status |
|----------------|---------------|--------|--------|
| 0-10% | +28.89% | 3,257 | ✅ Healthy |
| 10-20% | +11.58% | 1,694 | ✅ Acceptable |
| 20-30% | **-10.05%** | 211 | ⚠️ Marginal |
| 30-40% | **-19.44%** | 211 | ❌ At Risk |
| 40-50% | **-35.71%** | 326 | ❌ Loss Territory |
| 50%+ | **-119.20%** | 685 | ❌ Major Loss |

**Recommendation:** Implement 29.9% discount cap → **$135,376 annual recovery**

### 2. RFM Customer Segmentation
9 segments identified with distinct behaviors:

| Segment | Customers | Profit | % of Total | Avg CLV | Strategy |
|---------|-----------|--------|------------|---------|----------|
| Champions | 85 (10.7%) | $60,341 | 21.1% | $5,357 | VIP treatment, exclusive offers |
| Loyal Customers | 72 (9.1%) | $51,688 | 18.1% | $5,172 | Loyalty rewards, early access |
| Potential Loyalists | 142 (17.9%) | $48,285 | 16.9% | $2,861 | Upsell campaigns, engagement |
| **At Risk** | 57 (7.2%) | $35,356 | 12.4% | $3,684 | **Win-back campaigns (urgent)** |
| Hibernating | 59 (7.4%) | $28,012 | 9.8% | $3,581 | Reactivation offers |

**80/20 Rule:** Top 5 segments = 80% of profit

### 3. Customer Lifetime Value (CLV)
3-year projection model reveals value tiers:

- **Average CLV:** $3,633
- **Median CLV:** $2,584
- **90th Percentile (Platinum):** $6,948+
- **Recommended Max CAC:** $1,090 per customer (30% of avg CLV)

---

## 💼 Business Recommendations

### Immediate Actions (Week 1)
1. **Discount Cap Implementation**
   - Hard cap: 29.9% maximum
   - Require VP approval for 20-29.9% range
   - **Expected Impact:** +$135,376 annual profit

2. **At Risk Segment Rescue**
   - 57 customers, $35,356 at stake
   - Send win-back offers within 48 hours
   - **Target:** Recover 30% = $10,607

### Medium-Term Strategy (Month 1-3)
3. **Month 1 Retention Campaign**
   - **Problem:** 87.9% churn in first month
   - **Target:** Lift to 20% retention (+63 repeat customers)
   - **Expected Revenue:** +$162,000

4. **VIP Program Launch**
   - Target: Platinum tier (90th percentile = 80 customers)
   - Benefits: Dedicated rep, loyalty discount
   - **Justification:** $6,948 avg CLV supports premium service

---

## 🛠️ Technical Implementation

### Tech Stack
```python
# Data Manipulation
pandas (2.0+)          # DataFrames, aggregations
numpy (1.24+)          # Numerical operations

# Visualization
matplotlib (3.7+)      # Static charts
seaborn (0.13+)        # Statistical plots
plotly (5.18+)         # Interactive dashboards

# Analysis Methods
- RFM Segmentation (quintile-based scoring)
- Customer Lifetime Value (AOV × frequency × lifespan)
- Cohort Analysis (retention heatmaps)
- Pareto Analysis (80/20 rule)
```

### Key Algorithms

**RFM Scoring**
```python
RScore = pd.qcut(Recency, q=5, labels=[5,4,3,2,1])
FScore = pd.qcut(Frequency, q=5, labels=[1,2,3,4,5])
MScore = pd.qcut(Monetary, q=5, labels=[1,2,3,4,5])
```

**CLV Calculation**
```python
CLV_3Y = AOV × (Orders/Lifespan_Days × 365) × 3
```

---

## 🚀 How to Run

### Google Colab (Recommended)
1. Open `Superstore_Analysis-27.ipynb` in Colab
2. Upload `Sample - Superstore.csv` when prompted
3. Run All Cells (Ctrl+F9)
4. Dashboards auto-generate in ~30 seconds

### Local Jupyter
```bash
git clone https://github.com/MdZeeshan-ML/superstore-profitability-analysis.git
cd superstore-profitability-analysis
pip install -r requirements.txt
jupyter notebook Superstore_Analysis-27.ipynb
```

### Requirements
```txt
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.13.0
plotly>=5.18.0
```

---

## 📖 Data Dictionary

### Source Data (9,994 transactions)
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `OrderDate` | datetime | Transaction date | 2016-11-08 |
| `CustomerId` | string | Unique customer ID | CG-12520 |
| `Sales` | float | Revenue per order | $261.96 |
| `Profit` | float | Net profit/loss | $41.91 |
| `Discount` | float | Discount rate (0-1) | 0.00 (0%) |
| `Category` | string | Product type | Furniture |
| `Segment` | string | Customer type | Consumer/Corporate/Home Office |

### Calculated Fields
| Field | Formula | Purpose |
|-------|---------|---------|
| `RFMSegment` | f(RScore, FScore, MScore) | Champions, Loyal, At Risk, etc. |
| `CLV3Y` | AOV × Annual Orders × 3 | 3-year revenue projection |
| `Profitability` | Profit / Sales | Margin percentage |

---

## 🎓 Skills Demonstrated

### Data Analysis
✅ Cleaned 9,994 records (zero nulls)  
✅ Created 7 calculated metrics  
✅ Segmented 793 customers into 9 groups  
✅ Projected 3-year CLV for entire customer base

### Statistical Methods
✅ RFM Segmentation (quintile-based)  
✅ CLV Modeling (time-adjusted frequency)  
✅ Cohort Analysis (retention tracking)  
✅ Pareto Analysis (80/20 profit concentration)

### Visualization
✅ Executive dashboard (8-panel layout)  
✅ Interactive Plotly charts (hover, zoom, filter)  
✅ Retention heatmaps  
✅ Portfolio analysis

### Business Translation
✅ $135K profit quantification  
✅ ROI calculation (773% Year 1)  
✅ Segment-specific action plans  
✅ Prioritized recommendations by impact

---

## 📜 Certifications

<div align="center">

| Certificate | Issuer | Credential |
|-------------|--------|-----------|
| **Python (Basic)** | HackerRank | [5b5e02831c73](https://www.hackerrank.com/certificates/5b5e02831c73) |
| **SQL (Basic)** | HackerRank | [9e8c92208cbc](https://www.hackerrank.com/certificates/9e8c92208cbc) |
| **SQL (Intermediate)** | HackerRank | [95324fb8b70b](https://www.hackerrank.com/certificates/95324fb8b70b) |

</div>

---

## 🤝 Let's Connect

I'm available for freelance data analysis projects, especially:
- Retail/e-commerce profitability analysis
- Customer segmentation and CLV modeling
- Discount/pricing optimization
- Retention and churn analysis
- Python automation for data pipelines

📧 **Email:** ig.zeeshad@gmail.com  
💼 **HackerRank:** [@Blanc_coder](https://www.hackerrank.com/Blanc_coder) (5⭐ Python, 3⭐ Problem Solving)  


---

## ⭐ Support This Project

If you found this analysis helpful:
1. ⭐ Star this repository
2. 🔄 Share with your network
3. 📧 Reach out for collaboration

---

**Built with 💻 by Mohammad Zeeshan Hussain | January 2026**
