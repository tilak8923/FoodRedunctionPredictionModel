# Exploratory Data Analysis (EDA) - Step-by-Step Guide
## Food Waste Reduction using Predictive Analytics

---

## What is EDA?

**EDA (Exploratory Data Analysis)** is the process of:
- Understanding your data before modeling
- Finding patterns, trends, and anomalies
- Deciding what features to use
- Identifying data quality issues

**In simple words:** Before you predict something, you must understand what you're predicting!

---

## Your Dataset Overview

| Column | Meaning |
|--------|---------|
| **date** | Date of sale (2013-01-01 to 2017-12-31) |
| **store** | Store ID (1-10) |
| **item** | Product ID (1-50) |
| **sales** | Units sold that day |

**Total Records:** 913,000+

---

## EDA Steps Explained

### **STEP 1: Import & Load Data**

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv('train.csv')
```

**What you're doing:**
- Loading the CSV file into a pandas DataFrame (like an Excel sheet in Python)
- Getting all visualization & math libraries ready

**Why it matters:** You need these tools to explore and visualize data.

---

### **STEP 2: First Look (Head, Info, Stats)**

```python
df.head()          # First 5 rows
df.info()          # Column types and non-null counts
df.describe()      # Mean, std, min, max, quartiles
```

**What to check:**
- ✅ Are column names clear?
- ✅ Are data types correct? (dates should be datetime, not string)
- ✅ What's the min/max for each column?

**Example output:**
```
date: 2013-01-01 to 2017-12-31 ✓
store: 1 to 10 ✓
item: 1 to 50 ✓
sales: 0 to 100+ ← Some days have zero sales!
```

---

### **STEP 3: Missing Values**

```python
df.isnull().sum()
```

**What it does:** Counts how many empty cells (NaN) are in each column.

**Why it matters:** 
- Missing values cause model errors
- Need to handle them (delete rows, fill with average, etc.)

**Good news for your data:** No missing values! ✓

---

### **STEP 4: Create Time Features**

```python
df['date'] = pd.to_datetime(df['date'])
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['dayofweek'] = df['date'].dt.dayofweek  # 0=Mon, 6=Sun
```

**Why?**
- Raw dates are hard to use in models
- Breaking them into pieces (month, day) helps models learn patterns

**Example:**
- "Fridays might have higher sales than Mondays"
- "December might have different sales than July"

---

### **STEP 5: Sales Distribution**

```python
plt.hist(df['sales'], bins=50)
plt.show()
```

**What to look for:**
- Is the distribution normal (bell curve)? Or skewed?
- Are there outliers (very high/low sales)?
- What's the most common sales value?

**For your data:**
- Most days have sales between 5-20 units
- Some days have 0 sales (stockouts or closed?)
- Some days have 50+ sales (unusual spikes)

**Implication for waste:**
- Days with zero sales: no waste (good!)
- Days with high sales: accurate forecasting crucial (prevent loss)

---

### **STEP 6: Time Series Trend**

```python
daily_total = df.groupby('date')['sales'].sum()
plt.plot(daily_total)
```

**What it shows:**
- How total sales change over **years**
- Is demand growing? Stable? Declining?

**What to look for:**
- 📈 Upward trend = business growing
- 📉 Downward trend = losing market share
- 🔄 Flat = stable demand

**Why it matters:**
- If demand is growing 5% per year, models must account for this
- Outdated historical data might not predict future well

---

### **STEP 7: Day-of-Week Seasonality** ⭐ IMPORTANT

```python
df.groupby('dayofweek')['sales'].mean()
```

**What it shows:**
- Average sales for each day (Mon=0, Tue=1, ..., Sun=6)

**Real-world example:**
```
Monday:    10.2 units
Tuesday:   10.5 units
...
Friday:    12.1 units  ← HIGHEST
Saturday:  11.8 units
Sunday:    9.3 units   ← LOWEST
```

**Why it matters for waste reduction:**
- Order more on Fridays (expected demand higher)
- Order less on Sundays (lower demand = less spoilage risk)
- Prevents overstocking on low-demand days

---

### **STEP 8: Monthly Seasonality**

```python
df.groupby('month')['sales'].mean()
```

**Example pattern:**
```
January:   9.5 units   (cold, fewer people shopping)
...
December:  13.2 units  (holidays, gift buying)
```

**Why it matters:**
- Plan inventory based on expected monthly patterns
- December might need 40% more stock than January

---

### **STEP 9: Store Analysis**

```python
df.groupby('store')['sales'].mean()
```

**What you might find:**
```
Store 1: 9.2 units (downtown, high traffic)
Store 2: 11.5 units (mall location, busiest)
Store 3: 7.1 units (neighborhood, quiet)
...
```

**Implication:**
- Each store should have custom forecasts
- A model trained on all stores combined = less accurate
- Store 2 needs more stock; Store 3 needs less

---

### **STEP 10: Item Analysis**

```python
df.groupby('item')['sales'].mean().sort_values(ascending=False)
```

**What you find:**
```
Item 1:  12.3 units (popular - milk, bread, etc.)
Item 2:  11.8 units
...
Item 47: 0.8 units  (slow-moving item)
Item 50: 0.6 units  (rarely bought)
```

**Implication:**
- Item 1: Easy to forecast, high waste risk if overstocked
- Item 50: Hard to forecast (unpredictable), still matters
- Popular items = better forecasting = bigger impact on waste reduction

---

### **STEP 11: Store × Item Interaction**

```python
pivot = df.pivot_table(values='sales', index='item', columns='store')
# Creates a matrix: rows=items, cols=stores
```

**Example insight:**
```
Item 1 (milk):
  Store 1 (downtown): 15 units/day
  Store 2 (mall):     18 units/day
  Store 3 (suburb):   8 units/day

Item 45 (specialty):
  Store 1: 2 units/day
  Store 2: 3 units/day
  Store 3: 0.5 units/day
```

**Why it matters:**
- Some items sell well in certain stores, poorly in others
- Item 1 sells 2x better at Store 2 than Store 3
- One global forecast = wrong for everyone!

---

### **STEP 12: Year-over-Year Trend**

```python
df.groupby('year')['sales'].mean()
```

**Example:**
```
2013: 10.1 units/day
2014: 10.3 units/day
2015: 10.5 units/day
2016: 10.7 units/day
2017: 11.0 units/day
```

**Insight:** Slight upward trend (3% growth per year)

**For forecasting:**
- 2018 demand might be ~11.3 units/day on average
- But you still need seasonal adjustments

---

### **STEP 13: Outliers** ⚠️ IMPORTANT

```python
Q1 = df['sales'].quantile(0.25)
Q3 = df['sales'].quantile(0.75)
IQR = Q3 - Q1
outliers = df[(df['sales'] < Q1 - 1.5*IQR) | (df['sales'] > Q3 + 1.5*IQR)]
```

**What outliers are:**
- Data points that don't fit the normal pattern
- Example: 60 units sold when average is 10

**Why they happen:**
- Promotional event (spike)
- System error (false zero)
- Store closure (incomplete data)
- Seasonal surge

**For your project:**
- **Keep them:** More realistic, captures rare events
- **Remove them:** Cleaner model, less noise
- **Handle them:** Flag as anomalies, separate forecasting

**Recommendation:** Keep them for a 4-week semester project (shows you can handle real data!)

---

### **STEP 14: Correlation**

```python
df[['store', 'item', 'sales', 'month', 'dayofweek']].corr()
```

**Example output:**
```
store      → 0.15  (weak - store choice slightly affects sales)
item       → 0.32  (moderate - some items sell more than others)
month      → 0.08  (weak - seasonal but not dominant)
dayofweek  → 0.25  (moderate - day of week matters)
```

**What it means:**
- Values close to 1: Strong positive relationship
- Values close to -1: Strong negative relationship
- Values close to 0: No relationship

**Why it matters:**
- Which features should your model focus on?
- Item matters more (0.32) than month (0.08)
- Don't overweight weak features

---

### **STEP 15: Summary & Insights**

After all steps, you should write down:

1. **What are the main patterns?**
   - E.g., "Sales are 20% higher on Fridays"

2. **Which factors matter most?**
   - E.g., "Store location and item type are more important than month"

3. **What are the challenges?**
   - E.g., "Item 45 is unpredictable; Store 3 has high volatility"

4. **What should the model do?**
   - E.g., "Predict daily sales per store-item combo to optimize inventory"

---

## Common EDA Questions & Answers

### Q: Why should we visualize instead of just looking at numbers?

**A:** Humans understand pictures faster than tables.

```
Numbers:      [10.1, 10.3, 10.5, 10.7, 11.0]  ← Is there a trend? 
Chart:        📈 (obvious!)
```

### Q: What if the data has lots of zeros?

**A:** 
- Check: Are stores closed on certain days?
- Solution: Create a "zero sales" flag as a feature
- Model separately: Predict (a) whether item sells today, (b) how many if it does

### Q: Do I need to understand every detail?

**A:** No, but understand the big picture:
- ✅ What am I predicting? (Sales)
- ✅ What data do I have? (Daily sales by store & item)
- ✅ What patterns exist? (Day-of-week, seasonality, growth)
- ✅ What challenges exist? (Outliers, unpredictable items)

---

## Output: What to Keep for Your Report

After EDA, save these for your final report:

1. **Data Summary Table**
   - Rows, columns, date range, key statistics

2. **4-5 Key Visualizations**
   - Time series trend
   - Day-of-week pattern
   - Store comparison
   - Top items
   - One more interesting finding

3. **Key Insights List**
   - 5-7 bullet points of main findings
   - How they relate to waste reduction

4. **Data Quality Assessment**
   - Missing values: None ✓
   - Outliers: ~X%
   - Data span: 5 years ✓
   - Ready for modeling: Yes ✓

---

## Next Steps

Once you complete EDA:

1. ✅ **Commit findings to report** (2-3 pages of Data section)
2. ✅ **Feature engineering** (create additional features for models)
3. ✅ **Train predictive models** (Linear Regression, Random Forest, ARIMA)
4. ✅ **Compare & evaluate** (which model is best?)
5. ✅ **Estimate waste reduction** (if forecasts are accurate, how much waste is prevented?)

---

## Tips for Running the Notebook

1. **Run cells one by one** (not all at once)
2. **Read the output** before moving to next cell
3. **Modify visualizations** if needed (change colors, sizes)
4. **Take screenshots** of interesting plots (for your report)
5. **Add your own questions** and explore further!

---

**Good luck with your EDA! This is the most important step.** 🚀
