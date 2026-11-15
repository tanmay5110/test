# DMV Practicals 1–7: Line-by-Line Explanation

Every block has two voices:
- **Easy:** what it means in plain words
- **Technical:** what actually happens inside pandas / numpy / matplotlib / sklearn

> Note: in Practical 1 (Steps 3–5) and Practical 6 (Step 1) the code cells were not visible in your PDFs, only their outputs. I explained those parts by reading the outputs and warnings. They are marked **[reconstructed]**.

---

# PRACTICAL 1: Analyzing Sales Data from Multiple File Formats

**Goal:** Real data comes in different formats (CSV, Excel, JSON). Load all of them, clean them, merge into one table, compute revenue and plot it.

## Cell 1: Creating sample datasets

```python
import pandas as pd
import numpy as np
import json
import matplotlib.pyplot as plt
import seaborn as sns
```
- `pandas as pd`: library for tables (DataFrames). *Easy:* Excel inside Python. *Technical:* `pd` is the conventional alias; DataFrame = 2D labelled array.
- `numpy as np`: numeric arrays and `np.nan` (missing value).
- `json`: built-in module to read/write JSON text.
- `matplotlib.pyplot as plt`: base plotting library.
- `seaborn as sns`: nicer statistical plots on top of matplotlib.

```python
csv_data = {
    'Transaction_ID': [101, 102, 103, 104, 105, 105],
    'Product': ['Laptop','Mouse','Monitor','Keyboard','Laptop','Laptop'],
    'Price': [1200, 25, 300, 80, 1200, 1200],
    'Quantity': [1, 2, np.nan, 1, 1, 1],
    'Date': ['2023-10-01', ... ]
}
```
- A Python **dictionary**: key = column name, value = list of column values.
- `105` appears twice with identical values: a **deliberate duplicate** so you can practise duplicate removal.
- `np.nan` in Quantity: a **deliberate missing value** to practise imputation.
- Dates are written as **strings**, not real dates yet, so you can practise type conversion.

```python
pd.DataFrame(csv_data).to_csv('sales_data.csv', index=False)
```
- `pd.DataFrame(dict)` converts the dict to a table.
- `.to_csv(...)` writes it to a file.
- `index=False` stops pandas from saving the 0,1,2… row numbers as an extra column.

```python
excel_data = {... 'Product': ['mouse','Tablet','Monitor'] ...}
pd.DataFrame(excel_data).to_excel('sales_data.xlsx', index=False)
```
- Same idea, saved as Excel. Needs `openpyxl` installed (Colab has it).
- `'mouse'` is **lowercase on purpose**: an inconsistency (Mouse vs mouse) to fix later.

```python
json_data = [ {...}, {...} ]
with open('sales_data.json', 'w') as f:
    json.dump(json_data, f)
```
- JSON data = **list of dictionaries**, each dict = one row (typical API format).
- `with open(..., 'w') as f`: opens a file for writing; `with` closes it automatically even if an error happens.
- `json.dump(obj, f)` writes Python object as JSON text into the file.

```python
print("Sample files generated successfully!")
```
- Just a confirmation message.

## Cell 2: Steps 1 and 2, Load and explore

```python
df_csv = pd.read_csv('sales_data.csv')
df_excel = pd.read_excel('sales_data.xlsx')
df_json = pd.read_json('sales_data.json')
```
- Each reader parses a different format into the **same structure**: a DataFrame. After this, format doesn't matter.

```python
df_csv.info()
```
- `.info()` prints: row count, column names, non-null counts, dtypes, memory.
- **Reading the output:** Quantity has `5 non-null` out of 6, so 1 missing; it is `float64` (not int) because NaN can't live in an int column; `Date` is `object` (= string).

## Steps 3–5 **[reconstructed from warnings/outputs]**: Merge and clean

Likely code:
```python
df_combined = pd.concat([df_csv, df_excel, df_json], ignore_index=True)
df_cleaned = df_combined.drop_duplicates(subset='Transaction_ID')
df_cleaned['Quantity'] = df_cleaned['Quantity'].fillna(1)
df_cleaned['Product']  = df_cleaned['Product'].str.capitalize()
df_cleaned['Total_Revenue'] = df_cleaned['Price'] * df_cleaned['Quantity']
df_cleaned['Date'] = pd.to_datetime(df_cleaned['Date'])
```
- `pd.concat([...])`: stack the three tables vertically (rows added). `ignore_index=True` renumbers the index 0..10.
- `drop_duplicates()`: removes repeated rows. In the output, index 5 is **missing** (0,1,2,3,4,6,7…), proving the duplicate row 5 was dropped.
- `fillna(1)`: fills missing Quantity with 1. *Logic:* a sold transaction has at least 1 item; it's a business assumption. (Mean/median would give a silly 1.4.)
- `.str.capitalize()`: `mouse` to `Mouse`. Without this, "mouse" and "Mouse" would be grouped as 2 products.
- `Total_Revenue = Price * Quantity`: **feature engineering**, a new column from existing ones (vectorised; no loop needed).
- `pd.to_datetime`: string to real datetime so you can sort, group, plot by date.
- **Why the `SettingWithCopyWarning`:** `df_cleaned` came from a slice/filter of another DataFrame, so pandas isn't sure whether you're editing a copy or the original. **Fix:** `df_cleaned = df_combined.drop_duplicates(...).copy()`.

## Step 6: Descriptive statistics

```python
total_sales = df_cleaned['Total_Revenue'].sum()
avg_order_value = df_cleaned['Total_Revenue'].mean()
product_sales = df_cleaned.groupby('Product')['Total_Revenue'].sum().reset_index()
```
- `.sum()` = total revenue = **$5,445**. `.mean()` = average order value = 5445/10 = **$544.50**.
- `groupby('Product')`: split rows into groups by product. `['Total_Revenue'].sum()`: add up revenue inside each group. `reset_index()`: turns the group labels back into a normal column so the result is a clean DataFrame.
- **Split-Apply-Combine** is the idea: split by key, apply a function, combine results.

```python
print(f"Total Revenue Generated: ${total_sales:,.2f}")
```
- **f-string**: `{}` inserts a variable. `:,.2f` = thousands comma + 2 decimals (`5,445.00`).
- `display(...)` is a Jupyter function that shows a DataFrame as a pretty table (`print` would show plain text).

## Step 7: Visualizations

```python
sns.set_theme(style="whitegrid")
plt.figure(figsize=(10, 5))
```
- Sets a clean white background with gridlines. `figsize=(width, height)` in inches.

```python
plt.subplot(1, 2, 1)
sns.barplot(data=product_sales, x='Product', y='Total_Revenue', palette='viridis')
```
- `subplot(rows, cols, position)`: 1 row, 2 columns, draw in the 1st. Lets two charts sit side by side.
- `barplot`: bar chart. `palette='viridis'` = colour scheme.
- **FutureWarning:** newer seaborn wants `hue='Product', legend=False` when using a palette.

```python
plt.title(...); plt.xlabel(...); plt.ylabel(...)
```
- Chart title and axis labels (always label your axes).

```python
plt.subplot(1, 2, 2)
trend_df = df_cleaned.groupby('Date')['Total_Revenue'].sum().reset_index()
sns.lineplot(data=trend_df, x='Date', y='Total_Revenue', marker='o', color='b')
plt.xticks(rotation=45)
```
- Group by date to get **revenue per day**, then line chart (best for time trends). `marker='o'` shows dots at data points. `rotation=45` tilts date labels so they don't overlap.

```python
plt.tight_layout()
plt.show()
```
- `tight_layout()` auto-fixes overlapping spacing. `show()` renders the figure.

**Insight from output:** Laptop dominates revenue ($3600). Daily revenue is spiky because high-price laptop sales happen on only a few days.

---

# PRACTICAL 2: Weather Data via API (with Simulated Fallback)

**Goal:** Fetch weather forecast JSON from an API, flatten it into a table, aggregate daily, visualise, and find correlations.

## 1. Setup

```python
import requests
from datetime import datetime, timedelta
sns.set_theme(style="whitegrid")
plt.rcParams["figure.figsize"] = (10, 6)
API_KEY = "YOUR_API_KEY_HERE"
CITY_NAME = "London"
```
- `requests`: library to make HTTP calls to web APIs.
- `datetime` = date/time objects; `timedelta` = a duration (e.g. 3 hours).
- `plt.rcParams[...]`: sets a **global default** figure size so you don't repeat it.
- API key = your password for the OpenWeatherMap service. Placeholder, so the real call fails (401).

## 2. Fetching data with fallback

```python
def get_weather_data(city, api_key):
    url = f"http://api.openweathermap.org/data/2.5/forecast?q={city}&appid={api_key}&units=metric"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"API request failed with status: {response.status_code}. Generating simulated data instead...")
        return generate_mock_data()
```
- `def` defines a reusable function.
- The URL is built with an f-string: `q=` city, `appid=` key, `units=metric` for °C.
- `requests.get(url)` sends a GET request. `status_code`: 200 = OK, 401 = unauthorized (bad key), 404 = not found.
- `.json()` converts response text into Python dict/list.
- **Fallback pattern:** if the API fails, use fake data so the pipeline still runs. This is robust design.

```python
def generate_mock_data():
    np.random.seed(42)
    base_time = datetime.now()
    list_data = []
    for i in range(40):
        timestamp = base_time + timedelta(hours=i * 3)
        list_data.append({
            "dt": int(timestamp.timestamp()),
            "main": {"temp": round(np.random.normal(18, 5), 2),
                     "humidity": int(np.random.uniform(40, 95))},
            "wind": {"speed": round(np.random.uniform(1, 10), 2)},
            "rain": {"3h": round(np.random.exponential(0.5), 2) if np.random.rand() > 0.7 else 0.0}
        })
    return {"list": list_data, "city": {...}}
```
- `np.random.seed(42)`: makes random numbers **reproducible** (same output every run).
- `range(40)` x 3 hours = **120 hours = 5 days**, matching the real API (5-day / 3-hour forecast).
- `timestamp.timestamp()`: converts datetime to **Unix time** (seconds since 1970), which the real API uses in `dt`.
- `np.random.normal(18, 5)`: temperature from a bell curve (mean 18, std 5). `uniform(40, 95)`: any value equally likely in range. `exponential(0.5)`: right-skewed, good for rainfall (mostly small, rarely large).
- `if np.random.rand() > 0.7 else 0.0`: only ~30% of slots have rain, the rest 0.
- The **nested dict structure** (`main → temp`) copies the real API's JSON shape exactly, so downstream code works for both.

```python
raw_data = get_weather_data(CITY_NAME, API_KEY)
```
- Calls the function. Output "401" confirms fallback was used.

## 3. Extraction and preprocessing

```python
extracted_records = []
for entry in raw_data["list"]:
    record = {
        "datetime": datetime.fromtimestamp(entry["dt"]),
        "temperature": entry["main"]["temp"],
        "humidity": entry["main"]["humidity"],
        "wind_speed": entry["wind"]["speed"],
        "precipitation": entry.get("rain", {}).get("3h", 0.0)
    }
    extracted_records.append(record)
```
- **Flattening nested JSON** into one flat dict per row.
- `fromtimestamp` converts Unix seconds back to readable datetime.
- `entry.get("rain", {}).get("3h", 0.0)`: **safe access**. The real API omits the `rain` key when it isn't raining; `entry["rain"]` would crash with KeyError. `.get(key, default)` returns the default instead.

```python
df = pd.DataFrame(extracted_records)
df["date"] = df["datetime"].dt.date
```
- List of dicts to DataFrame (each dict = row).
- `.dt.date` extracts only the date part (drops time), used for **daily grouping**.

## 4. Aggregation

```python
daily_summary = df.groupby("date").agg(
    avg_temp=("temperature", "mean"),
    max_temp=("temperature", "max"),
    min_temp=("temperature", "min"),
    avg_humidity=("humidity", "mean"),
    avg_wind_speed=("wind_speed", "mean"),
    total_precipitation=("precipitation", "sum")
).reset_index()
```
- **Named aggregation**: `new_name=(column, function)`. Computes many stats per day in one go.
- Temperature: mean/max/min make sense. Rain: **sum** (total daily rain), not mean.
- 6 rows = 6 calendar dates (partial first and last days).

## 5. Dual-axis visualization

```python
fig, ax1 = plt.subplots(figsize=(12, 6))
sns.lineplot(data=df, x="datetime", y="temperature", ax=ax1, color="tab:orange", label="Temperature (°C)")
ax1.tick_params(axis="y", labelcolor="tab:orange")
ax2 = ax1.twinx()
sns.lineplot(data=df, x="datetime", y="precipitation", ax=ax2, color="tab:blue", ...)
ax2.fill_between(df["datetime"], df["precipitation"], color="tab:blue", alpha=0.3)
```
- `subplots()` returns a figure (`fig`) and axes (`ax1`).
- `ax1.twinx()`: creates a **second y-axis sharing the same x-axis**. Needed because °C (≈10-25) and mm (≈0-1) have totally different scales.
- Colour-coded axis labels (orange/blue) tell the reader which axis belongs to which line.
- `fill_between(x, y)`: shades the area under rain line. `alpha=0.3` = 30% opacity.
- The comment in code says "lineplot instead of barplot" because bar charts on datetime axes cause coordinate mismatch.

## 6. Correlation analysis

```python
correlation_matrix = df[["temperature","humidity","wind_speed","precipitation"]].corr()
sns.heatmap(correlation_matrix, annot=True, cmap="coolwarm", fmt=".2f", linewidths=0.5)
```
- `.corr()` = **Pearson correlation** between every pair, range −1 to +1.
- `heatmap` colours the matrix; `annot=True` writes the numbers; `fmt=".2f"` 2 decimals; `coolwarm` red=positive, blue=negative.
- **Reading result:** humidity–wind = 0.33 (weak positive), temperature–precipitation = −0.29 (weak negative). Data is random, so all correlations are weak. In real data, humidity and rain would correlate more.

---

# PRACTICAL 3: Telecom Customer Churn: Data Cleaning and Preparation

**Goal:** Take a raw customer dataset and produce **model-ready** train/test files for predicting churn (customers who leave).

## 1. Import and explore

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
url = 'https://raw.githubusercontent.com/IBM/.../Telco-Customer-Churn.csv'
df = pd.read_csv(url)
print("Dataset Shape:", df.shape)
display(df.head()); display(df.info())
```
- `read_csv` can read directly from a URL.
- `.shape` = (7043 rows, 21 columns). `.head()` = first 5 rows. `.info()` = types and nulls.
- Observed: `TotalCharges` is `object` (text) even though it should be a number. This is a hidden data-quality problem.

## 2. Missing values and data types

```python
df['TotalCharges'] = df['TotalCharges'].replace(' ', np.nan)
df['TotalCharges'] = pd.to_numeric(df['TotalCharges'])
```
- Blank cells in this dataset are a **single space** `' '`, not true NaN, which is why the column became text.
- Replace with NaN, then `to_numeric` converts the whole column to float.

```python
print(df.isnull().sum())
```
- `isnull()` gives True/False grid; `.sum()` counts True per column.

```python
if df['TotalCharges'].isnull().any():
    median_val = df['TotalCharges'].median()
    df['TotalCharges'] = df['TotalCharges'].fillna(median_val)
```
- `.any()` = is there at least one missing? If yes, fill with **median**.
- *Why median:* robust to outliers. Mean gets pulled by very high values.

```python
categorical_cols = df.select_dtypes(include=['object']).columns.difference(['customerID'])
for col in categorical_cols:
    df[col] = df[col].astype('category')
```
- Select all text columns **except** customerID (an identifier, not a feature).
- `category` dtype = memory-efficient for repeated text values and signals "this is a category".

## 3. Remove duplicates

```python
duplicates_count = df.duplicated().sum()
if duplicates_count > 0:
    df = df.drop_duplicates()
```
- `duplicated()` flags repeated rows. Found 0 here. (customerID is unique anyway.)

## 4. Standardize inconsistent values

```python
for col in df.select_dtypes(include=['category']).columns:
    df[col] = df[col].str.strip()
    df[col] = df[col].replace({'No internet service': 'No', 'No phone service': 'No'})
    df[col] = df[col].astype('category')
display(df.describe(include='all'))
```
- `.str.strip()` removes stray leading/trailing spaces.
- Dataset uses 3 values (Yes / No / "No internet service") for service columns. Merging the third into "No" gives a clean **binary** feature, since "no internet" effectively means no add-on.
- `astype('category')` again because `.str` operations turn it back to object.
- `describe(include='all')` summarises numeric and categorical columns together; NaN shown where a stat doesn't apply (e.g. mean of text).
- *Careful:* this loop also hits the `InternetService` column, which legitimately has "No". Its 3 unique values remain (DSL / Fiber optic / No).

## 5. Outliers (IQR capping)

```python
for col in ['tenure','MonthlyCharges','TotalCharges']:
    Q1 = df[col].quantile(0.25)
    Q3 = df[col].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    df[col] = np.clip(df[col], lower_bound, upper_bound)
```
- Q1 = 25th percentile, Q3 = 75th. **IQR = Q3 − Q1** = spread of the middle 50%.
- Anything beyond **1.5 × IQR** from the quartiles is an outlier (Tukey's rule, same rule boxplots use).
- `np.clip` **caps** (winsorises) values to the bounds instead of deleting rows, so no data is lost.

## 6. Feature engineering

```python
df['Charges_to_Tenure_Ratio'] = df['MonthlyCharges'] / (df['tenure'] + 1)
df['Churn'] = df['Churn'].map({'Yes': 1, 'No': 0})
```
- New feature = monthly bill per month of tenure. **+1 avoids division by zero** for brand-new customers (tenure = 0).
- `.map({...})` converts the target to numbers (ML models need numeric targets).

## 7. Scaling and encoding

```python
X = df.drop(columns=['customerID', 'Churn'])
y = df['Churn']
X = pd.get_dummies(X, drop_first=True)
```
- **X = features, y = target (label)**. Drop the ID because it carries no pattern.
- `get_dummies` = **one-hot encoding**: a column with k categories becomes k−1 True/False columns.
- `drop_first=True` avoids the **dummy variable trap** (perfect multicollinearity: if all but one are known, the last is implied). Result: 24 columns.

```python
scaler = StandardScaler()
numerical_features = ['tenure','MonthlyCharges','TotalCharges','Charges_to_Tenure_Ratio']
X[numerical_features] = scaler.fit_transform(X[numerical_features])
```
- **Standardisation:** z = (x − mean) / std, giving mean 0, std 1. Stops big-number features (TotalCharges in thousands) from dominating small ones.
- `fit_transform` = learn mean/std, then apply.
- *Better practice:* split first, `fit` on train only, `transform` on test, to avoid **data leakage**. Here scaling happens before the split.

## 8. Train-test split

```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
```
- 80% train (5634) / 20% test (1409).
- `random_state=42`: reproducible split.
- `stratify=y`: keeps the same churn percentage in both sets (important because churners are a minority, about 26%).

## 9. Export

```python
X_train.to_csv('X_train.csv', index=False) ...
```
- Saves four files for use in later modelling notebooks.

---

# PRACTICAL 4: Data Wrangling on Real Estate Market

**Goal:** Practise wrangling steps: clean column names, missing values, filter, encode, aggregate, outliers.
(The dataset loaded is a UK house-price time series, so some later cells print "column not found".)

## 1. Import and clean column names

```python
url = 'https://raw.githubusercontent.com/datasets/house-prices-uk/master/data/data.csv'
try:
    df = pd.read_csv(url)
except Exception:
    df = pd.read_csv('RealEstate_Prices.csv')
```
- `try/except`: if the URL fails (no internet, link dead), fall back to a local file instead of crashing.

```python
df.columns = df.columns.str.strip().str.lower().str.replace(' ', '_').str.replace('[^a-zA-Z0-9_]', '', regex=True)
```
- Chained **string cleaning on column names**: strip spaces, lowercase, spaces to underscores, remove special characters (regex `[^a-zA-Z0-9_]` = anything not letter/digit/underscore).
- Why: consistent names like `price_all` are easy to type, no spaces, no case errors.

## 2. Missing values

```python
for col in df.select_dtypes(include=[np.number]).columns:
    df[col] = df[col].fillna(df[col].median())
for col in df.select_dtypes(include=['object']).columns:
    df[col] = df[col].fillna(df[col].mode()[0])
```
- **Numeric columns → median. Text columns → mode** (most frequent value; `[0]` takes the first if there are ties).
- Final check `isnull().sum().sum()` = 0 (first `.sum()` per column, second adds them up).
- *Side effect:* in the output, the first rows' `change_all` shows 6.6 where it was NaN. The first row has no previous period, so a "change" doesn't truly exist; median-filling invents data. For time series, forward-fill/leave NaN is more accurate.

## 3. Filter and subset

```python
if 'property_type' in df.columns:
    subset_df = df[df['property_type'].isin(['Residential','Single Family'])]
else:
    subset_df = df.copy()
```
- `.isin([...])` = keep rows matching any listed value (boolean masking).
- Column doesn't exist in this dataset, so it takes the `else` and copies everything. `.copy()` makes an independent table.

## 4. Categorical encoding

```python
categorical_cols = subset_df.select_dtypes(include=['object']).columns.tolist()
encoded_df = pd.get_dummies(subset_df, columns=categorical_cols, drop_first=True)
```
- Only text column is `date`, so **every unique date becomes its own column**, giving **297 columns**.
- *Lesson:* never one-hot encode a date or high-cardinality column. Convert with `pd.to_datetime` and extract year/month/quarter instead.

## 5. Aggregate

```python
if 'neighborhood' in df.columns and 'sale_price' in df.columns:
    neighborhood_summary = df.groupby('neighborhood')['sale_price'].agg(['mean','median','count']).reset_index()
```
- Would give per-neighborhood mean, median, count. Guard `if` prevents a KeyError; printed message shows the columns are missing.

## 6. Outliers

```python
Q1, Q3, IQR, lower_bound, upper_bound ...
cleaned_df = df[(df['sale_price'] >= lower_bound) & (df['sale_price'] <= upper_bound)]
```
- Same IQR rule as Practical 3, but here outliers are **removed** (filtered out) rather than capped.
- `&` = element-wise AND; each condition needs its own parentheses.
- Again skipped because `sale_price` doesn't exist.

---

# PRACTICAL 5: Air Quality Index (AQI) Trends

**Goal:** Visualise time-series and relationships in air-quality data.

## Dummy data

```python
np.random.seed(42)
dates = pd.date_range(start='2023-01-01', periods=100, freq='D')
pm25 = np.random.normal(loc=35, scale=15, size=100).clip(5, 150)
pm10 = pm25 * 1.5 + np.random.normal(loc=10, scale=5, size=100)
co = np.random.uniform(0.1, 2.5, size=100)
aqi = pm25 * 1.8 + np.random.normal(loc=20, scale=10, size=100)
aqi = np.clip(aqi, 10, 300)
```
- `pd.date_range(..., freq='D')`: 100 consecutive daily dates.
- PM2.5 from normal distribution, `.clip(5,150)` forces realistic limits.
- **PM10 depends on PM2.5** (×1.5 + noise) and **AQI depends on PM2.5** (×1.8 + 20 + noise). The data is made *correlated on purpose* so the scatter plot later shows a clear trend. CO is independent.

```python
df_mock = pd.DataFrame({'Date': dates, 'PM2.5': pm25, ...})
df_mock.to_csv('City_Air_Quality.csv', index=False)
```
- Build table and save as CSV (simulating a downloaded dataset).

## Load and explore

```python
df = pd.read_csv('City_Air_Quality.csv')
df['Date'] = pd.to_datetime(df['Date'])
```
- CSV stores dates as text; convert back to datetime64.

## Overall AQI trend

```python
plt.plot(df['Date'], df['AQI'], color='#2ca02c', linewidth=2)
plt.grid(True, linestyle='--', alpha=0.5)
```
- Basic line plot. `#2ca02c` = hex colour (green). Dashed light grid.
- The x-axis labels overlap because the figure is small. Fix: `plt.xticks(rotation=45)`.

## Individual pollutants

```python
fig, axs = plt.subplots(3, 1, figsize=(7, 4), sharex=True)
axs[0].plot(df['Date'], df['PM2.5'], ...)
```
- 3 rows, 1 column of stacked charts. `sharex=True` = all share the same time axis. Use `axs[i].set_title()` (axes methods) instead of `plt.title()` when there are multiple axes.

## Monthly average

```python
df['Month'] = df['Date'].dt.strftime('%Y-%m')
monthly_aqi = df.groupby('Month')['AQI'].mean().reset_index()
plt.bar(monthly_aqi['Month'], monthly_aqi['AQI'], color='#9467bd', width=0.5)
```
- `strftime('%Y-%m')` formats a date as text like `2023-01` (year-month). It is used as the grouping key.
- Mean AQI per month, shown as a bar chart (categories compared).

## AQI category boxplot

```python
def get_category(aqi_val):
    if aqi_val <= 50: return 'Good'
    elif aqi_val <= 100: return 'Moderate'
    elif aqi_val <= 150: return 'Unhealthy for Sensitive Groups'
    else: return 'Unhealthy'
df['Category'] = df['AQI'].apply(get_category)
data_to_plot = [df[df['Category'] == cat]['AQI'] for cat in categories]
plt.boxplot(data_to_plot, labels=categories, patch_artist=True, boxprops=dict(facecolor='#bcbd22', color='black'))
```
- Follows the US EPA AQI bands.
- `.apply(func)` runs your function on every value (row-by-row mapping).
- **List comprehension**: builds a list of AQI-value arrays, one per category.
- **Boxplot** shows median (orange line), box (Q1–Q3), whiskers (range) and outliers, per category.
- 'Unhealthy' box is empty because the max AQI in this data is about 135 (< 150).
- `patch_artist=True` allows filling boxes with colour. The `labels` argument is deprecated, so the warning says to use `tick_labels`.

## AQI vs pollutants (scatter)

```python
scatter = plt.scatter(df['PM2.5'], df['AQI'], c=df['PM10'], cmap='viridis', s=df['CO']*100, alpha=0.7)
cbar = plt.colorbar(scatter)
cbar.set_label('PM10 Level')
```
- A **4-variable chart**: x = PM2.5, y = AQI, **colour = PM10**, **size = CO** (×100 so points are visible).
- `colorbar` = legend for the colour scale. `alpha=0.7` handles overlapping points.
- The clear upward diagonal = strong positive correlation (PM2.5 drives AQI).

---

# PRACTICAL 6: Retail Sales Performance Analysis

**Goal:** Analyse sales by region and by region × product category.

## Import / Load **[Load cell not visible, reconstructed]**
```python
import pandas as pd, matplotlib.pyplot as plt, seaborn as sns
df = pd.read_csv('retail_sales.csv')   # (not shown) must have: region, product_category, sales_amount
```
- The later cells use `df`, so a loading cell must have run. Columns used: `region`, `product_category`, `sales_amount`.

## 2. Aggregate by region

```python
region_sales = df.groupby('region')['sales_amount'].sum().reset_index()
region_sales = region_sales.sort_values(by='sales_amount', ascending=False)
```
- Total sales per region, then sorted high to low. Index labels (3,1,0,2) show the original order is preserved after sorting.
- Result: West 125,510 > North 115,285 > East 107,875 > South 107,330.

## 3. Bar chart

```python
plt.figure(figsize=(5, 3))
sns.barplot(data=region_sales, x='region', y='sales_amount', palette='viridis')
plt.xticks(rotation=45)
```
- Sorted bars make ranking obvious at a glance. FutureWarning about `palette` same as Practical 1.

## 4. Top region

```python
top_region = region_sales.iloc[0]
print(f"... {top_region['region']} ... ${top_region['sales_amount']:.2f}")
```
- Since the table is sorted descending, **row 0 = best**. `.iloc[0]` selects a row by *position* (integer location). The row (a Series) is then indexed by column name.

## 5. Region × category

```python
category_sales = df.groupby(['region','product_category'])['sales_amount'].sum().reset_index()
```
- **Group by two keys**, producing 4 regions × 3 categories = 12 rows.

## 6. Stacked bar chart

```python
pivot_sales = category_sales.pivot(index='region', columns='product_category', values='sales_amount')
pivot_sales.plot(kind='bar', stacked=True, figsize=(5,4), colormap='tab20')
plt.legend(title='Product Category', bbox_to_anchor=(1.05, 1), loc='upper left')
```
- `pivot` reshapes **long to wide**: rows = regions, columns = categories, cells = sales. This is the shape needed for stacked plotting.
- `kind='bar', stacked=True`: each bar's segments = categories, so you see the total **and** the composition.
- `bbox_to_anchor=(1.05,1)`: moves legend outside the plot so it doesn't cover the bars.
- Insight: North's biggest category is Electronics (46,400); East's is Clothing (42,880).

---

# PRACTICAL 7: Stock Market Time Series Analysis

**Goal:** Load stock prices, analyse trend, moving averages, seasonality, correlation, then forecast with ARIMA.
(Heading says Netflix but the code actually uses Apple (AAPL) from the S&P 500 dataset.)

## 1. Dataset acquisition

```python
from statsmodels.tsa.seasonal import seasonal_decompose
from statsmodels.tsa.arima.model import ARIMA
import warnings, kagglehub, os
warnings.filterwarnings('ignore')
path = kagglehub.dataset_download("camnugent/sandp500")
csv_path = os.path.join(path, 'all_stocks_5yr.csv')
full_df = pd.read_csv(csv_path)
df = full_df[full_df['Name'] == 'AAPL'].copy()
```
- `statsmodels` = statistics/time-series library.
- `warnings.filterwarnings('ignore')` hides warnings (clean output, though it can hide useful info).
- `kagglehub.dataset_download` downloads a Kaggle dataset and returns the folder path.
- `os.path.join` builds a file path that works on any OS.
- The file holds **all 500 companies**; the filter `Name == 'AAPL'` keeps only Apple rows (1259 trading days, about 5 years). `.copy()` avoids SettingWithCopyWarning.

```python
df = df.rename(columns={'date':'Date','open':'Open',...})
```
- Renames lowercase to Capitalised, because the later code expects those names.

## 2. Explore and datetime formatting

```python
df['Date'] = pd.to_datetime(df['Date'])
df.set_index('Date', inplace=True)
df.sort_index(inplace=True)
```
- Text to datetime.
- `set_index('Date')`: **makes Date the row label**, required for time-series functions like `resample` and `rolling`.
- `sort_index` ensures chronological order. `inplace=True` modifies the table directly (no reassignment).

## 3. Historical trend

```python
plt.plot(df['Close'], color='royalblue')
```
- Plotting a Series with a datetime index automatically puts dates on the x-axis. Shows an uptrend from about $60 (2013) to about $170 (2018).
- **Close** price = price at market closing, the standard analysis column.

## 4. Moving averages

```python
df['MA30'] = df['Close'].rolling(window=30).mean()
df['MA100'] = df['Close'].rolling(window=100).mean()
```
- `rolling(30).mean()` = average of the **last 30 days**, sliding forward one day at a time. First 29 values are NaN (not enough history).
- MA30 = short-term trend (follows price closely); MA100 = long-term trend (smoother, lags).
- Traders use crossovers: short MA crossing above long MA = bullish ("golden cross"), below = bearish ("death cross").

## 5. Seasonality analysis

```python
weekly_close = df['Close'].resample('W').mean().ffill()
decomposition = seasonal_decompose(weekly_close, model='additive', period=52)
fig = decomposition.plot()
```
- `resample('W').mean()`: converts daily data to **weekly averages**, reducing noise.
- `.ffill()`: forward-fill any gap weeks with the last known value.
- `seasonal_decompose` splits the series into **Trend + Seasonal + Residual** (additive: Y = T + S + R).
- `period=52` = 52 weeks in a year, so it looks for patterns repeating **yearly**.
- Output: four panels (Observed / Trend / Seasonal / Residual). Residual is large and structured, meaning yearly seasonality is weak for stocks; the pattern is mostly trend.
- Trend line is shorter at the ends because the centred moving average used to estimate it needs data on both sides.

## 6. Correlation

```python
correlation_matrix = df[['Open','High','Low','Close','Volume']].corr()
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', fmt='.2f')
```
- Open/High/Low/Close are ≈ **1.00 correlated** (they all move together within a day). Volume is about −0.62 with prices: higher prices coincided with lower trading volume in this period.
- For modelling this means **multicollinearity**: don't feed all four price columns to a regression.

## 7. ARIMA forecasting

```python
model = ARIMA(weekly_close, order=(5, 1, 0))
results = model.fit()
forecast_steps = 26
forecast = results.get_forecast(steps=forecast_steps)
forecast_index = pd.date_range(start=weekly_close.index[-1], periods=forecast_steps + 1, freq='W')[1:]
forecast_series = pd.Series(forecast.predicted_mean.values, index=forecast_index)
```
- **ARIMA(p, d, q)** = AutoRegressive Integrated Moving Average.
  - `p = 5`: uses the last 5 weeks' values to predict the next.
  - `d = 1`: **differences once** (predicts weekly *change* instead of price) to make the series stationary.
  - `q = 0`: no moving-average error terms.
- `fit()` estimates parameters. `get_forecast(26)` predicts 26 weeks ≈ 6 months.
- `date_range(... periods=27)[1:]` builds future weekly dates, skipping the first (the last known date).
- `.predicted_mean.values` = forecast numbers; wrapped into a Series with those future dates.
- Plot: last 104 weeks (`[-104:]`, 2 years) as a solid line + forecast as a **red dashed line**.
- The forecast is nearly flat: ARIMA with d=1 and no trend term basically predicts "about where we are now". It's a baseline; stock prices are close to a random walk and are hard to forecast.

---

# CONCEPTS USED IN EACH PRACTICAL (Summary)

| Practical | Main concepts |
|---|---|
| 1 | Multi-format ingestion (CSV/Excel/JSON), `concat`, duplicate removal, imputation, text standardisation, feature engineering, `groupby`, bar and line plots, subplots |
| 2 | REST API, HTTP status codes, JSON parsing/flattening, fallback and error handling, simulation with random distributions, named aggregation, dual-axis plot, correlation heatmap |
| 3 | Data cleaning pipeline, type correction, median imputation, IQR outlier capping, feature engineering, one-hot encoding, standardisation, stratified train-test split |
| 4 | Column-name normalisation, try/except fallback, imputation (median/mode), boolean filtering, one-hot encoding (and its pitfall), IQR outlier removal, defensive `if` checks |
| 5 | Time-series line plots, subplots with shared axis, resampling by month, rule-based categorisation (`apply`), boxplot, multi-variable scatter, correlation |
| 6 | Multi-key `groupby`, sorting, `iloc`, pivot table, stacked bar chart |
| 7 | Datetime index, rolling mean, resampling, seasonal decomposition, correlation / multicollinearity, ARIMA forecasting |

---

# CONCEPT GLOSSARY (Easy + Technical)

**DataFrame / Series.** *Easy:* a table / one column of a table. *Technical:* pandas 2D labelled structure / 1D labelled array.

**CSV / Excel / JSON.** CSV = plain text with commas; Excel = binary spreadsheet; JSON = nested key-value text (APIs). All read into DataFrames via `read_csv / read_excel / read_json`.

**API.** *Easy:* a website door for programs. *Technical:* HTTP endpoint returning (usually) JSON; check `status_code` (200 OK, 401 unauthorized, 404 not found).

**Missing values (NaN).** Empty cells. Handle by dropping, or **imputing**: mean (symmetric data), median (skewed/outliers), mode (categories), constant (business rule), forward-fill (time series).

**Duplicates.** Repeated records that inflate counts/sums. `duplicated()` / `drop_duplicates()`.

**Data type conversion.** `astype`, `to_numeric`, `to_datetime`. Wrong types block calculation (text can't be summed) or time operations.

**Standardising text.** `str.strip/lower/capitalize/replace` so "mouse" = "Mouse".

**Outliers.** Extreme values. **IQR rule:** outside [Q1 − 1.5·IQR, Q3 + 1.5·IQR]. Handle by **removing** (filtering) or **capping/winsorising** (`np.clip`). Other method: z-score (|z| > 3).

**Feature engineering.** Creating new useful columns (Total_Revenue, ratios, month from date).

**GroupBy (split-apply-combine) and aggregation.** Split rows by key, apply sum/mean/count, combine. `agg` allows several functions at once.

**Pivot.** Long to wide reshape (rows × columns matrix).

**Boolean filtering / `isin`.** Keep rows where a condition is True; combine with `&`, `|` and parentheses.

**Encoding.** ML needs numbers. **One-hot** = a column per category (`get_dummies`). `drop_first=True` avoids dummy trap. **Label/map encoding** = Yes/No to 1/0. Avoid one-hot on high-cardinality columns (like dates).

**Scaling.** **StandardScaler** (z-score: mean 0, std 1) vs **MinMaxScaler** (0 to 1). Needed for distance/gradient-based models. Fit on train only to avoid leakage.

**Train-test split.** Hold out data to evaluate honestly. `test_size`, `random_state` (reproducibility), `stratify` (keep class ratio).

**Data leakage.** Information from test data influencing training (e.g. scaling before the split).

**Descriptive statistics.** `describe()`, mean, median, std, quantiles, count.

**Correlation.** Pearson r ∈ [−1, 1]: +1 move together, −1 opposite, 0 no linear relation. *Correlation ≠ causation.* High correlation between features = multicollinearity.

**Visualization choices.** Line = trend over time; Bar = compare categories; Stacked bar = total + composition; Boxplot = distribution/outliers/median; Scatter = relationship between 2 (+2 via colour/size) variables; Heatmap = matrix of values (correlation); Dual-axis = two different units on one time axis.

**Matplotlib vs Seaborn.** Matplotlib = low-level control; Seaborn = high-level, prettier statistical plots built on matplotlib. `subplot / subplots` = multiple charts in a grid; `tight_layout` = avoid overlap.

**Time-series concepts.**
- *Datetime index:* needed for time operations.
- *Resampling:* change frequency (daily to weekly/monthly).
- *Rolling window / moving average:* smooths noise to show trend.
- *Decomposition:* Observed = Trend + Seasonality + Residual (additive) or product (multiplicative).
- *Stationarity:* mean/variance don't change over time; differencing (`d`) helps achieve it.
- *ARIMA(p,d,q):* AR (past values), I (differencing), MA (past errors). Good as a baseline; stocks are close to a random walk.

**Python concepts used.** Dictionaries & lists, functions (`def`), loops, list comprehension, f-strings, `try/except`, `if/else` guards, `with open(...)` context manager, random seed for reproducibility, `inplace`, method chaining.

**Warnings explained.**
- `SettingWithCopyWarning`: you modified a slice; use `.copy()` or `.loc`.
- `FutureWarning` (seaborn palette): pass `hue=` as well.
- `MatplotlibDeprecationWarning` (`labels` to `tick_labels`): argument renamed.

---

# QUICK VIVA / EXAM POINTS

1. Why `index=False` while saving? Avoid an extra index column in the file.
2. Why median not mean? Robust to outliers.
3. Why `drop_first=True`? Prevent multicollinearity (dummy variable trap).
4. Why scale features? Equal contribution of features; many models are sensitive to scale.
5. Why `stratify`? Keep class proportions the same in train and test.
6. Why `random_state` / `seed`? Reproducible results.
7. Why a fallback to simulated data? Keeps the pipeline running when the API is unavailable.
8. Why resample to weekly for ARIMA/decomposition? Reduces noise and makes the yearly period 52 (not 252 trading days).
9. Why ARIMA `d=1`? Remove trend so the series becomes stationary.
10. Why IQR 1.5? Tukey's convention; matches the whiskers of a boxplot.
