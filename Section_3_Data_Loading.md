# 📊 SECTION 3: DATA LOADING & PREPROCESSING

## Purpose
Loads the Superstore dataset, performs essential data cleaning, creates derived features, and validates data quality. This is the foundation for all subsequent analysis.

---

## 📋 Cell Structure

### **Cell 5 - Load Data (Code)**
**What it does:**
- Imports CSV file
- Converts date columns to proper datetime format
- Extracts date components (year, month, day)
- Creates calculated fields (profitability ratio)
- Validates data quality

### **Cell 6 - Data Overview (Markdown)**
**What it contains:**
- Dataset statistics summary
- Date range and scope
- Key column descriptions

---

## 💻 Code Template

### Cell 5: Data Loading Code

```python
# Load and preprocess data
df = pd.read_csv('Sample - Superstore.csv', encoding='ISO-8859-1')

# Date conversion
df['Order_Date'] = pd.to_datetime(df['Order Date'], format='%m/%d/%Y')
df['Ship_Date'] = pd.to_datetime(df['Ship Date'], format='%m/%d/%Y')

# Extract date components
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

---

### Cell 6: Data Overview Markdown

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

## 🔍 Code Explanation

### Step 1: Import Data
```python
df = pd.read_csv('Sample - Superstore.csv', encoding='ISO-8859-1')
```
**Purpose:** Load CSV into pandas DataFrame  
**Encoding:** ISO-8859-1 handles special characters in customer names/addresses  
**Alternative:** If file is in different location, use full path:
```python
df = pd.read_csv('/path/to/Sample - Superstore.csv', encoding='ISO-8859-1')
```

### Step 2: Date Conversion
```python
df['Order_Date'] = pd.to_datetime(df['Order Date'], format='%m/%d/%Y')
df['Ship_Date'] = pd.to_datetime(df['Ship Date'], format='%m/%d/%Y')
```
**Purpose:** Convert string dates to datetime objects  
**Why:** Enables time-based operations (grouping by month, calculating days between dates)  
**Format:** `'%m/%d/%Y'` matches "12/31/2017" format  

**Troubleshooting:**
- If dates are in "YYYY-MM-DD" format, use: `format='%Y-%m-%d'`
- For automatic detection (slower): `pd.to_datetime(df['Order Date'], infer_datetime_format=True)`

### Step 3: Extract Date Components
```python
df['OrderY'] = df['Order_Date'].dt.year
df['OrderM'] = df['Order_Date'].dt.month
df['OrderD'] = df['Order_Date'].dt.day
```
**Purpose:** Create separate columns for year, month, day  
**Use Cases:**
- `OrderY`: Year-over-year comparisons
- `OrderM`: Seasonal analysis (monthly trends)
- `OrderD`: Day-of-month patterns (payday effects)

**Additional Options:**
```python
df['OrderDOW'] = df['Order_Date'].dt.dayofweek  # 0=Monday, 6=Sunday
df['OrderQ'] = df['Order_Date'].dt.quarter      # 1-4 quarters
df['OrderWeek'] = df['Order_Date'].dt.isocalendar().week  # Week of year
```

### Step 4: Calculated Fields
```python
df['Profitability'] = df['Profit'] / df['Sales']
```
**Purpose:** Create profit margin ratio (0 to 1)  
**Interpretation:**
- 0.15 = 15% profit margin
- Negative value = loss-making transaction

**Additional Calculated Fields:**
```python
# Shipping delay in days
df['Ship_Delay'] = (df['Ship_Date'] - df['Order_Date']).dt.days

# Revenue per unit (if Quantity available)
df['Price_Per_Unit'] = df['Sales'] / df['Quantity']

# Cost of goods sold
df['COGS'] = df['Sales'] - df['Profit']
```

### Step 5: Data Validation
```python
print(f"✅ Loaded {len(df):,} orders")
print(f"📊 Date Range: {df['Order_Date'].min().date()} to {df['Order_Date'].max().date()}")
print(f"👥 Customers: {df['Customer ID'].nunique():,}")
print(f"💰 Revenue: ${df['Sales'].sum():,.2f}")
```
**Purpose:** Quick sanity check of loaded data  
**Expected Output:**
```
✅ Loaded 9,994 orders
📊 Date Range: 2014-01-03 to 2017-12-30
👥 Customers: 793
💰 Revenue: $2,297,200.86
```

---

## 🛠️ Data Quality Checks

### Advanced Validation (Optional)

Add this code to Cell 5 for comprehensive quality checks:

```python
# Check for missing values
print("\n🔍 Missing Values:")
print(df.isnull().sum()[df.isnull().sum() > 0])

# Check for duplicates
print(f"\n🔁 Duplicate Orders: {df.duplicated(subset=['Order ID']).sum()}")

# Check for negative values (potential data errors)
print(f"\n⚠️ Negative Sales: {(df['Sales'] < 0).sum()}")
print(f"⚠️ Negative Quantities: {(df['Quantity'] < 0).sum()}")

# Check discount range (should be 0-1)
print(f"\n🎯 Discount Range: {df['Discount'].min():.2f} to {df['Discount'].max():.2f}")
if df['Discount'].max() > 1:
    print("⚠️ Warning: Discounts > 100% detected")

# Data type summary
print("\n📊 Data Types:")
print(df.dtypes)
```

**Expected Healthy Output:**
```
🔍 Missing Values:
(No output = no missing values)

🔁 Duplicate Orders: 0

⚠️ Negative Sales: 0
⚠️ Negative Quantities: 0

🎯 Discount Range: 0.00 to 0.80
```

---

## 📊 Dataset Schema

### Core Transaction Fields
| Column | Type | Description | Example |
|--------|------|-------------|--------|
| Order ID | String | Unique order identifier | CA-2017-152156 |
| Order Date | DateTime | Purchase date | 2017-11-08 |
| Ship Date | DateTime | Shipment date | 2017-11-11 |
| Ship Mode | String | Shipping method | Standard Class |
| Customer ID | String | Unique customer identifier | CG-12520 |
| Customer Name | String | Full name | Claire Gute |
| Segment | String | Customer category | Consumer |

### Product Fields
| Column | Type | Description | Example |
|--------|------|-------------|--------|
| Category | String | Product category | Furniture |
| Sub-Category | String | Product subcategory | Bookcases |
| Product ID | String | SKU identifier | FUR-BO-10001798 |
| Product Name | String | Item description | Bush Somerset Collection |

### Geographic Fields
| Column | Type | Description | Example |
|--------|------|-------------|--------|
| Country | String | Always "United States" | United States |
| City | String | City name | Henderson |
| State | String | State name | Kentucky |
| Postal Code | Integer | ZIP code | 42420 |
| Region | String | Geographic region | South |

### Financial Fields
| Column | Type | Description | Example |
|--------|------|-------------|--------|
| Sales | Float | Revenue ($) | 261.96 |
| Quantity | Integer | Units sold | 2 |
| Discount | Float | Discount rate (0-1) | 0.00 |
| Profit | Float | Margin ($) | 41.91 |

---

## 🚨 Common Issues & Solutions

### Issue 1: File Not Found
**Error:** `FileNotFoundError: [Errno 2] No such file or directory`

**Solutions:**
```python
# Option 1: Upload to Google Colab
from google.colab import files
uploaded = files.upload()
df = pd.read_csv('Sample - Superstore.csv', encoding='ISO-8859-1')

# Option 2: Load from Google Drive
from google.colab import drive
drive.mount('/content/drive')
df = pd.read_csv('/content/drive/MyDrive/Sample - Superstore.csv', encoding='ISO-8859-1')

# Option 3: Load from URL
url = 'https://raw.githubusercontent.com/username/repo/main/Sample - Superstore.csv'
df = pd.read_csv(url, encoding='ISO-8859-1')
```

### Issue 2: Date Parsing Errors
**Error:** `ValueError: time data '01/03/2014' does not match format '%Y-%m-%d'`

**Solution:**
```python
# Let pandas infer the format
df['Order_Date'] = pd.to_datetime(df['Order Date'], infer_datetime_format=True)

# Or use errors='coerce' to convert unparseable dates to NaT
df['Order_Date'] = pd.to_datetime(df['Order Date'], errors='coerce')
print(f"Invalid dates: {df['Order_Date'].isna().sum()}")
```

### Issue 3: Encoding Errors
**Error:** `UnicodeDecodeError: 'utf-8' codec can't decode byte`

**Solution:**
```python
# Try different encodings
df = pd.read_csv('Sample - Superstore.csv', encoding='latin-1')  # or
df = pd.read_csv('Sample - Superstore.csv', encoding='cp1252')   # or
df = pd.read_csv('Sample - Superstore.csv', encoding='utf-8-sig')
```

### Issue 4: Column Name Mismatches
**Error:** `KeyError: 'Order Date'`

**Solution:**
```python
# Print all column names
print(df.columns.tolist())

# Standardize column names (remove spaces)
df.columns = df.columns.str.replace(' ', '_')
df['Order_Date'] = pd.to_datetime(df['Order_Date'])
```

---

## 📝 Markdown Cell 6: Detailed Version

For more comprehensive documentation:

```markdown
---

## 📋 DATASET OVERVIEW

### Data Source
**Origin:** Tableau Superstore Sample Dataset (via Kaggle)  
**Type:** Retail sales transactions  
**Geography:** United States  
**Industry:** Office supplies, furniture, technology  

### Dataset Statistics
- **Total Records:** 9,994 transactions
- **Date Range:** January 3, 2014 – December 30, 2017 (4 years)
- **Unique Customers:** 793
- **Unique Products:** 1,862 SKUs
- **Geographic Coverage:** 49 states, 531 cities

### Financial Summary
- **Total Revenue:** $2,297,200.86
- **Total Profit:** $286,397.02
- **Overall Margin:** 12.5%
- **Average Order Value:** $229.86

### Key Columns Reference

#### Identifiers
- `Order ID`: Unique per transaction (can have multiple line items)
- `Customer ID`: Unique per customer (enables cohort analysis)
- `Product ID`: Unique per SKU

#### Temporal
- `Order_Date`: When customer placed order
- `Ship_Date`: When order was shipped
- `OrderY`, `OrderM`, `OrderD`: Extracted date components

#### Financial
- `Sales`: Gross revenue (before discounts applied to profit calc)
- `Profit`: Net margin after COGS and discounts
- `Discount`: Percentage off (0 = no discount, 0.20 = 20% off)
- `Profitability`: Calculated profit margin ratio

#### Categorical
- `Segment`: Consumer / Corporate / Home Office
- `Category`: Furniture / Office Supplies / Technology
- `Sub-Category`: 17 subcategories (Chairs, Phones, Binders, etc.)
- `Region`: Central / East / South / West

### Data Quality Notes
- ✅ No missing values in critical fields
- ✅ No duplicate Order IDs
- ✅ All dates within expected range
- ⚠️ Some orders have negative profit (expected for loss leaders)
- ⚠️ Discount range: 0% to 80% (some very high discounts)
```

---

## 💡 Customization Options

### For Different Datasets

If you're adapting this for a different CSV:

```python
# Generic loading template
df = pd.read_csv('your_file.csv', encoding='utf-8')

# Detect date columns automatically
date_columns = df.select_dtypes(include=['object']).columns[
    df.select_dtypes(include=['object']).apply(pd.to_datetime, errors='coerce').notna().any()
]

for col in date_columns:
    df[col] = pd.to_datetime(df[col], errors='coerce')
    print(f"Converted {col} to datetime")

# Auto-create year/month from first date column
first_date_col = date_columns[0]
df['Year'] = df[first_date_col].dt.year
df['Month'] = df[first_date_col].dt.month
```

### For Large Datasets (> 1M rows)

```python
# Read in chunks to save memory
chunk_size = 100000
chunks = []

for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # Process each chunk
    chunk['Order_Date'] = pd.to_datetime(chunk['Order Date'])
    chunks.append(chunk)

df = pd.concat(chunks, ignore_index=True)
print(f"Loaded {len(df):,} rows in chunks")
```

---

## ✅ Validation Checklist

Before proceeding to analysis sections:

- [ ] Data loaded successfully (no errors)
- [ ] Row count matches expected (~9,994 for Superstore)
- [ ] Date columns are datetime type (not object/string)
- [ ] Order_Date, Ship_Date both converted
- [ ] OrderY, OrderM, OrderD columns created
- [ ] Profitability column calculated
- [ ] No critical missing values
- [ ] Date range is reasonable (2014-2017)
- [ ] Revenue sum matches expected (~$2.3M)

---

## 🎯 Next Steps
After completing this section:
1. Run Cell 5 to load data
2. Verify output matches expected statistics
3. Proceed to Section 4 (Discount Analysis) or Section 5 (RFM)
4. All subsequent sections depend on this data being loaded

---

**Section Status:** ✅ Template Complete  
**Dependencies:** None (first analysis section)  
**Estimated Runtime:** 1-2 seconds  
**Output Type:** Loaded DataFrame + validation prints
