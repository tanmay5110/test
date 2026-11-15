# DMV Master Guide: Concepts + Graphs (Simple Language)

Covers everything used in Practicals 1–7. Each topic: **what it is → why → how → example → common mistake**.

---

# PART 0: THE BIG PICTURE

**DMV = working with data and showing it visually.** Every practical follows the same pipeline:

```
 1. GET data        → read CSV / Excel / JSON / API / Kaggle
 2. EXPLORE         → .head(), .info(), .describe(), .shape
 3. CLEAN           → missing values, duplicates, wrong types, typos, outliers
 4. TRANSFORM       → new columns, encoding, scaling, grouping, pivoting
 5. ANALYSE         → statistics, correlation, trends
 6. VISUALISE       → charts that answer a question
 7. (OPTIONAL) MODEL→ split data, forecast (ARIMA), predict (churn)
```

**Golden rule:** *Garbage in, garbage out.* A beautiful chart on dirty data is a beautiful lie.

---

# PART 1: DATA FUNDAMENTALS

## 1.1 Types of data

| Type | Meaning | Examples | Best chart |
|---|---|---|---|
| **Numerical, continuous** | any value in a range | price, temperature, AQI | histogram, line, scatter, boxplot |
| **Numerical, discrete** | whole counts | quantity, tenure (months) | bar, histogram |
| **Categorical, nominal** | labels, no order | product, region, gender | bar, pie |
| **Categorical, ordinal** | labels with order | AQI category: Good < Moderate < Unhealthy | ordered bar, boxplot |
| **Datetime** | points in time | order date, timestamp | line chart |
| **Identifier** | unique label, no meaning | customerID, Transaction_ID | never plot or model it |

**Why this matters:** the data type decides which operations and charts make sense. You can't average "Laptop", and you can't one-hot encode a date.

## 1.2 Structured data formats

- **CSV**: plain text, values separated by commas. Simple, universal, no data types stored (dates arrive as text).
- **Excel (.xlsx)**: spreadsheet, can have multiple sheets, formatting.
- **JSON**: nested "key: value" text. Used by APIs. A list of dictionaries = a table of rows.
- **API**: a program-to-program way to request data over the internet. Response codes: **200** OK, **401** unauthorized (bad key), **404** not found, **429** too many requests.

All become a **DataFrame** (a table) in pandas.

## 1.3 DataFrame anatomy

```
              columns (features)
            ┌──────────┬────────┬───────┐
   index →  │ Product  │ Price  │ Qty   │
   0        │ Laptop   │ 1200   │ 1     │  ← one row = one record/observation
   1        │ Mouse    │ 25     │ 2     │
            └──────────┴────────┴───────┘
```
- **Series** = one column. **Index** = row labels. **Shape** = (rows, columns).

---

# PART 2: DATA CLEANING (THE MOST IMPORTANT PART)

## 2.1 Missing values

**What:** empty cells (NaN = "Not a Number").
**Why they exist:** user didn't enter, sensor failed, merge mismatch, blank space stored as `' '`.

| Strategy | When | Example |
|---|---|---|
| Drop rows (`dropna`) | very few missing, random | lose 2 of 10,000 rows |
| **Mean** fill | numeric, symmetric, no outliers | exam marks |
| **Median** fill | numeric, skewed or has outliers | prices, salaries, TotalCharges |
| **Mode** fill | categorical | most frequent city |
| **Constant / business rule** | domain knowledge | missing Quantity = 1 (Practical 1) |
| **Forward-fill** (`ffill`) | time series | missing weekly price = last week's |

**Why median over mean?** Example: salaries 20k, 22k, 25k, 1,000k. Mean = 266k (nonsense); median = 23.5k (realistic).

**Mistake:** filling "change from last period" for the first row (Practical 4). The first row has no previous period, so a filled value is invented.

**Detect:** `df.isnull().sum()` → missing per column.

## 2.2 Duplicates
Same record repeated, so totals and counts are inflated. `df.duplicated().sum()`, `df.drop_duplicates()`. Define "duplicate" carefully: whole row, or just the ID (`subset='Transaction_ID'`).

## 2.3 Wrong data types
- Number stored as text (`TotalCharges` with blank `' '`) → fix with `pd.to_numeric`.
- Date stored as text → `pd.to_datetime`.
- Why it matters: text can't be summed; text dates sort alphabetically, not by time.

## 2.4 Inconsistent text
`mouse` vs `Mouse` vs ` Mouse ` count as 3 different products. Fix: `.str.strip()`, `.str.lower()`, `.str.capitalize()`, `.replace()`.
Category merging example: "No internet service" → "No".

## 2.5 Column-name cleaning
Lowercase, underscores, no special characters: `Price (USD)` → `price_usd`. Makes code easy and error-free.

## 2.6 Outliers

**What:** values far from the rest. Could be errors (typo: age 250) or real (billionaire's salary).

**Methods:**

1. **IQR rule (Tukey)**
   - Q1 = 25th percentile, Q3 = 75th, **IQR = Q3 − Q1**
   - Lower fence = Q1 − 1.5·IQR; upper fence = Q3 + 1.5·IQR
   - Outside the fences = outlier.
2. **Z-score:** z = (x − mean)/std; |z| > 3 = outlier (assumes bell-curve data).
3. **Visual:** boxplot dots beyond whiskers.

**What to do:**
- **Remove** rows (filter): when it's clearly an error. (Practical 4)
- **Cap / Winsorise** (`np.clip`): keep the row but pull the value to the fence. (Practical 3)
- **Keep:** if it's real and important (fraud, rare event).

**Percentile in plain words:** the 25th percentile is the value below which 25% of data lies.

---

# PART 3: TRANSFORMATION

## 3.1 Feature engineering
Creating new, more useful columns from existing ones.
- `Total_Revenue = Price × Quantity`
- `Charges_to_Tenure_Ratio = MonthlyCharges / (tenure + 1)` (+1 prevents divide-by-zero)
- `Month = date.dt.strftime('%Y-%m')`
- `Category = AQI value → Good/Moderate/...`

**Why:** models and charts often need the "right question" as a column.

## 3.2 GroupBy: Split → Apply → Combine

```
Raw rows                Split by Product        Apply sum            Combine
Laptop 1200             Laptop: 1200,1200,1200   3600                Laptop  3600
Mouse  50               Mouse:  50, 125          175                 Mouse    175
Laptop 1200             ...
```
- One key: `df.groupby('region')['sales'].sum()`
- Two keys: `df.groupby(['region','category'])['sales'].sum()`
- Many stats: `.agg(avg=('temp','mean'), mx=('temp','max'))`
- `reset_index()` turns the group labels back into a normal column.

**Pick the right aggregation:** temperature → mean/max/min; rainfall → **sum**; customers → count.

## 3.3 Pivot (long → wide)
```
LONG:                          WIDE (pivot):
East  Beauty   31225           region | Beauty | Clothing | Electronics
East  Clothing 42880           East   | 31225  | 42880    | 33770
...                            ...
```
Needed for stacked bars and heatmaps.

## 3.4 Sorting and selecting
- `sort_values(by=..., ascending=False)`: ranking.
- `.iloc[0]`: select by **position** (first row). `.loc[...]`: select by **label**.
- Boolean filter: `df[(df.price > 100) & (df.qty > 1)]` (use `&`, `|`, parentheses).
- `.isin([...])`: match any value in a list.

## 3.5 Merging tables
- `pd.concat([a, b, c])`: **stack** tables vertically (same columns).
- `pd.merge(a, b, on='key')`: **join** side by side by a key (like SQL JOIN).

## 3.6 Encoding categorical data (text → numbers)

ML models only understand numbers.

| Method | How | Use when |
|---|---|---|
| **Label / map** | Yes→1, No→0 | binary columns, target variable |
| **Ordinal** | Low→1, Med→2, High→3 | categories with order |
| **One-hot** | one True/False column per category | nominal categories (`get_dummies`) |

**`drop_first=True`:** with k categories you need only k−1 columns, because the last one is implied. Avoids the **dummy variable trap** (perfect multicollinearity).

**Mistake:** one-hot encoding a date or ID gives hundreds of columns (Practical 4 produced 297).

## 3.7 Scaling (making features comparable)

Problem: `tenure` ranges 0–72 but `TotalCharges` ranges 0–8000. Distance-based models would think TotalCharges matters 100× more.

- **StandardScaler (z-score):** (x − mean) / std → mean 0, std 1. Practical 3.
- **MinMaxScaler:** (x − min)/(max − min) → 0 to 1.
- Trees (Random Forest, XGBoost) don't need scaling; KNN, SVM, linear models, neural nets do.

## 3.8 Train-test split & leakage

- **Train (80%)**: model learns. **Test (20%)**: honest exam on unseen data.
- `random_state=42`: same split every run.
- `stratify=y`: keep class ratio (e.g. 26% churn) the same in both sets.
- **Data leakage:** test information sneaking into training (e.g. fitting the scaler on all data). Correct: `fit` on train, only `transform` on test.

---

# PART 4: STATISTICS YOU MUST KNOW

| Term | Plain meaning | Formula / pandas |
|---|---|---|
| Mean | average | `.mean()` |
| Median | middle value | `.median()` |
| Mode | most frequent | `.mode()` |
| Std deviation | typical spread around the mean | `.std()` |
| Variance | std² | `.var()` |
| Min / Max / Range | smallest / largest / difference | |
| Quartiles | 25%, 50%, 75% cut points | `.quantile(0.25)` |
| IQR | Q3 − Q1, spread of the middle half | |
| Skewness | lopsidedness (right tail = positive skew) | `.skew()` |
| Correlation | strength of linear relationship (−1 to +1) | `.corr()` |

`df.describe()` gives count, mean, std, min, 25%, 50%, 75%, max in one shot.

**Normal (bell) distribution:** ~68% of data within 1 std of the mean, ~95% within 2, ~99.7% within 3.

## Correlation in depth
- **+1:** perfectly together (PM2.5 and AQI ≈ high positive).
- **0:** no linear relation.
- **−1:** opposite directions.
- |r| < 0.3 weak, 0.3–0.7 moderate, > 0.7 strong (rule of thumb).
- **Correlation ≠ causation.** Ice-cream sales and drowning both rise in summer; one doesn't cause the other.
- **Multicollinearity:** two input features almost identical (Open/High/Low/Close ≈ 1.00). Confuses linear models; keep one or combine.
- Pearson only detects **linear** relationships. A perfect U-shape can have r ≈ 0.

---

# PART 5: GRAPHS: HOW TO READ AND WHEN TO USE EACH

## 5.0 Choosing the chart (decision guide)

| Your question | Use |
|---|---|
| How does X change over **time**? | **Line chart** |
| Compare values across **categories** | **Bar chart** |
| Compare categories **and** show what they're made of | **Stacked bar** |
| How is one numeric variable **distributed**? | **Histogram / boxplot** |
| Compare distributions across groups | **Boxplot** |
| Is there a **relationship** between two numbers? | **Scatter plot** |
| Relationships among **many** numbers at once | **Correlation heatmap** |
| Two measures with **different units**, same time axis | **Dual-axis line** |
| Show **parts of a whole** (few categories only) | Pie |
| Break a time series into parts | **Decomposition plot** |

## 5.1 Bar chart (Practicals 1, 5, 6)
- **Shows:** one value per category, bar height = value.
- **Read:** compare heights; sort bars descending for ranking.
- **Example:** Laptop revenue $3600 is far above Keyboard $320.
- **Rules:** bars must start at **0** (otherwise differences are exaggerated); label axes; sort when ranking.
- **Code:** `sns.barplot(data=, x=, y=)` / `plt.bar()`.

## 5.2 Line chart (Practicals 1, 2, 5, 7)
- **Shows:** a value over ordered points, usually time.
- **Read:** slope = growth/decline; spikes = events; wiggles = volatility.
- **Use markers (`marker='o'`)** when data points are few (daily revenue with 9 points).
- **Mistake:** line chart for unordered categories, which falsely implies a trend between them.

## 5.3 Stacked bar chart (Practical 6)
- **Shows:** total (full bar height) **and** composition (coloured segments).
- **Read:** West has the tallest bar (best total); North's blue/Electronics segment is biggest.
- **Weakness:** only the bottom segment starts from zero, so comparing middle segments across bars is hard.
- **Needs pivot** data (rows = regions, columns = categories).

## 5.4 Subplots (Practicals 1, 5)
- Several charts in one figure. `plt.subplot(rows, cols, position)` or `fig, axs = plt.subplots(3,1, sharex=True)`.
- `sharex=True` = same x-axis → easy vertical comparison across time.
- `plt.tight_layout()` stops overlaps.

## 5.5 Dual-axis (twin-axis) chart (Practical 2)
- **Shows:** two measures with different units/scales on one time axis (Temp °C left, Rain mm right).
- **Read:** colour-matched axis labels tell you which line uses which axis.
- **Caution:** can mislead because axis scales are arbitrary; use colours and clear labels.
- **Code:** `ax2 = ax1.twinx()`.

## 5.6 Heatmap / correlation matrix (Practicals 2, 7)
- **Shows:** a grid of numbers as colours. Here: correlation between every pair.
- **Read:** diagonal is always 1 (a variable with itself). Dark red = strong positive; dark blue = strong negative; pale = near 0. The matrix is symmetric (mirror across the diagonal).
- **Example:** Practical 7: Open/High/Low/Close ≈ 1.00 (same-day prices move together); Volume ≈ −0.62.
- **Code:** `sns.heatmap(corr, annot=True, cmap='coolwarm', fmt='.2f')`.

## 5.7 Boxplot (Practical 5)

```
        ┬  ← max (non-outlier)  "upper whisker"
        │
      ┌─┴─┐ ← Q3 (75%)
      │ ─ │ ← median (50%)
      └─┬─┘ ← Q1 (25%)
        │
        ┴  ← min (non-outlier)  "lower whisker"
        o  ← outlier (beyond 1.5×IQR)
```
- **Shows:** distribution summary: centre, spread, skew, outliers.
- **Read:** box height = IQR (the spread of the middle 50%); median off-centre in the box = skewed; whiskers uneven = skewed; dots = outliers; **empty slot** (the 'Unhealthy' category in Practical 5) = no data in that group.
- **Best for** comparing the same measure across groups side by side.

## 5.8 Scatter plot (Practical 5)
- **Shows:** each row as a dot at (x, y).
- **Read:** upward diagonal = positive relation; downward = negative; cloud = none; tight line = strong.
- **Extra dimensions:** colour (`c=PM10`) and size (`s=CO*100`) → 4 variables in one chart.
- **`alpha`** = transparency to see overlapping points. **Colorbar** = legend for colours.
- Practical 5: PM2.5 vs AQI is a tight rising band. This is strong positive correlation, because the data was generated that way.

## 5.9 Histogram (concept, not in the notebooks but essential)
- Counts how many values fall in each range (bin). Shows shape: bell-shaped, skewed, two peaks. `sns.histplot(df['col'], kde=True)`.

## 5.10 Time-series decomposition plot (Practical 7)
Four stacked panels:
1. **Observed**: the original weekly close.
2. **Trend**: slow long-term direction (up/down/flat).
3. **Seasonal**: pattern repeating every period (52 weeks = yearly).
4. **Residual**: leftover noise after removing trend and seasonality.

**Read:** if the residual is large or has structure, the model (trend + season) doesn't explain everything. For stocks, the trend dominates and seasonality is weak.

## 5.11 Moving-average chart (Practical 7)
- Light line = raw price (noisy); orange = 30-day MA; red = 100-day MA.
- **MA smooths noise.** Short MA hugs the price; long MA lags but shows the big trend.
- Price above long MA = uptrend. **Golden cross** = short MA crosses above long MA (bullish signal); **death cross** = crosses below.

## 5.12 Forecast plot (Practical 7)
- Solid line = history; dashed red = prediction. The nearly flat forecast shows ARIMA predicts "roughly stay near the last level", the best honest guess for random-walk-like stock prices.
- Real forecasts should also show a **confidence interval** (shaded band); none was drawn here.

## 5.13 Good-chart checklist (viva favourite)
1. Title that states what it shows.
2. Both axes labelled with units.
3. Legend when more than one series.
4. Sensible colours (consistent, colour-blind-friendly like `viridis`).
5. Bars start at zero.
6. No overlapping labels (rotate ticks).
7. Chart type matches the question.
8. Don't chart more than the viewer can read.

---

# PART 6: TIME-SERIES CONCEPTS (PRACTICALS 2, 5, 7)

| Concept | Simple meaning |
|---|---|
| **Time series** | data points in time order (stock price per day) |
| **Datetime index** | making the date the row label so time tools work |
| **Resampling** | change frequency: daily → weekly (`resample('W').mean()`) |
| **Rolling window** | slide a window of N points; compute average (moving average) |
| **Trend** | long-term direction |
| **Seasonality** | pattern repeating at fixed intervals (weekly, yearly) |
| **Noise / residual** | random leftover |
| **Additive model** | Y = Trend + Seasonal + Residual (changes are constant in size) |
| **Multiplicative model** | Y = Trend × Seasonal × Residual (changes scale with level) |
| **Stationary** | mean and variance stay constant over time |
| **Differencing** | subtract previous value to remove trend → stationary |
| **ARIMA(p,d,q)** | p = past values used, d = differencing times, q = past errors used |
| **Forecast horizon** | how far ahead we predict (26 weeks) |

**Stock prices ≈ random walk:** tomorrow's price ≈ today's + random change, so forecasts are flat and uncertain. Don't treat them as trading advice.

**Why resample before decomposing?** Daily data has gaps (weekends/holidays) and noise. Weekly averages are regular and smoother. `period=52` then means "one year".

---

# PART 7: PYTHON CONCEPTS USED

| Concept | Example | Meaning |
|---|---|---|
| Dictionary | `{'a': [1,2]}` | key → value; becomes DataFrame columns |
| List | `[101, 102]` | ordered collection |
| Function | `def f(x): return x*2` | reusable code block |
| f-string | `f"Total: {x:,.2f}"` | insert variables; `,` thousands, `.2f` 2 decimals |
| for loop | `for col in cols:` | repeat for each item |
| List comprehension | `[x for x in a if x>0]` | build a list in one line |
| try / except | `try: ... except: ...` | handle errors gracefully (fallback) |
| `with open()` | | auto-closes the file |
| Random seed | `np.random.seed(42)` | same "random" numbers every run |
| `inplace=True` | | modify the original object |
| Method chaining | `df.groupby().sum().reset_index()` | one step after another |
| Vectorisation | `df.a * df.b` | operates on whole columns, no loops, fast |
| `.apply(func)` | | run a function on each element/row |
| `.map(dict)` | | replace values using a dictionary |

**Random distributions:** `normal` (bell curve), `uniform` (equal chance in range), `exponential` (many small, few big; e.g. rainfall).

---

# PART 8: LIBRARIES AT A GLANCE

| Library | Role |
|---|---|
| **pandas** | tables, cleaning, grouping, time series |
| **numpy** | fast numeric arrays, NaN, random numbers, `clip` |
| **matplotlib** | base plotting, full control |
| **seaborn** | prettier statistical charts on top of matplotlib |
| **requests** | call web APIs |
| **json** | read/write JSON |
| **datetime** | dates, times, durations |
| **scikit-learn** | `train_test_split`, `StandardScaler`, ML models |
| **statsmodels** | time-series tools: `seasonal_decompose`, `ARIMA` |
| **kagglehub** | download Kaggle datasets |

---

# PART 9: COMMON WARNINGS & ERRORS

| Message | Meaning | Fix |
|---|---|---|
| `SettingWithCopyWarning` | you edited a slice of another DataFrame | `.copy()` or use `.loc` |
| `FutureWarning: palette without hue` | seaborn API change | add `hue='x', legend=False` |
| `KeyError: 'rain'` | key missing | use `.get('rain', {})` |
| `401` from API | bad/missing API key | use a real key (or fallback data) |
| `ValueError: could not convert string to float` | text in numeric column | find with `pd.to_numeric(errors='coerce')` |
| `KeyError: 'sale_price'` | column doesn't exist | check `df.columns`, add `if` guard |
| Overlapping x labels | too many/long ticks | `plt.xticks(rotation=45)` |

---

# PART 10: PRACTICAL-BY-PRACTICAL MAP

| # | Topic | Concepts | Charts |
|---|---|---|---|
| 1 | Sales, multi-format | read CSV/Excel/JSON, concat, dedupe, impute, text fix, revenue feature, groupby | bar + line (subplots) |
| 2 | Weather API | API/status codes, JSON flatten, `.get`, fallback, random simulation, named agg, correlation | dual-axis line, heatmap |
| 3 | Churn prep | type fix, median impute, IQR cap, feature eng, one-hot, scaling, stratified split | none (tables) |
| 4 | Real estate wrangling | column cleaning, try/except, impute, filter, encoding trap, outlier removal | none |
| 5 | AQI trends | date range, resample by month, categorisation, boxplot, multi-variable scatter | line, stacked subplots, bar, boxplot, scatter |
| 6 | Retail sales | groupby (1 and 2 keys), sort, iloc, pivot | bar, stacked bar |
| 7 | Stock time series | datetime index, rolling, resample, decomposition, correlation, ARIMA | line, MA lines, decomposition, heatmap, forecast |

---

# PART 11: INSIGHTS: HOW TO TALK ABOUT RESULTS

Always finish analysis with **what the chart/number tells you** (this is what examiners reward).

- **P1:** Laptops produce $3600 of $5445 total (66%). Revenue is spiky because high-price items sell on few days.
- **P2:** Temperature and rainfall weakly negatively correlated (−0.29): warmer slots had slightly less rain. Weak, as the data is simulated.
- **P3:** Dataset is ready for a classifier: 24 numeric features, 5634 train / 1409 test, class ratio preserved.
- **P5:** AQI is strongly driven by PM2.5; monthly averages are similar (≈ 77–86), March slightly higher; no 'Unhealthy' days.
- **P6:** West is the top region ($125,510); North is strongest in Electronics; East in Clothing.
- **P7:** AAPL shows a long-term uptrend, price stays above the 100-day MA during rallies; the ARIMA forecast is flat, so no strong directional signal.

**Template:** *"The [chart] shows [what]. We observe [pattern/number]. This suggests [meaning]. Limitation: [e.g. simulated data, small sample, correlation ≠ causation]."*

---

# PART 12: MASTER CHEAT SHEET

```python
# LOAD
pd.read_csv() / read_excel() / read_json()
df.head(); df.info(); df.describe(); df.shape; df.columns

# CLEAN
df.isnull().sum();  df.fillna(x);  df.dropna()
df.duplicated().sum();  df.drop_duplicates()
pd.to_numeric(col);  pd.to_datetime(col);  col.astype('category')
col.str.strip().str.lower().str.replace('a','b')
np.clip(col, lower, upper)

# TRANSFORM
df['new'] = df['a'] * df['b']
df.groupby('k')['v'].sum().reset_index()
df.groupby('k').agg(name=('col','mean'))
df.pivot(index=, columns=, values=)
pd.get_dummies(df, drop_first=True)
pd.concat([a, b], ignore_index=True)
df.sort_values('c', ascending=False)
df[df['c'].isin([...])];  df.iloc[0]

# TIME SERIES
df['Date'] = pd.to_datetime(df['Date']); df.set_index('Date')
s.rolling(30).mean();  s.resample('W').mean()
seasonal_decompose(s, model='additive', period=52)
ARIMA(s, order=(5,1,0)).fit().get_forecast(steps=26)

# ML PREP
train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)
StandardScaler().fit_transform(X)

# PLOT
plt.figure(figsize=(w,h)); plt.title(); plt.xlabel(); plt.ylabel()
plt.xticks(rotation=45); plt.grid(); plt.legend(); plt.tight_layout(); plt.show()
sns.barplot / lineplot / heatmap;  plt.plot / bar / scatter / boxplot
```

---

# PART 13: PRACTICE QUESTIONS (WITH SHORT ANSWERS)

1. **Difference between `concat` and `merge`?** concat stacks tables (rows or columns); merge joins by a key.
2. **Mean vs median imputation?** Median for skewed data/outliers.
3. **What is IQR and the outlier rule?** Q3 − Q1; outside Q1 − 1.5·IQR or Q3 + 1.5·IQR.
4. **Remove vs cap outliers?** Remove = delete rows; cap = clip to boundary, keeping data.
5. **Why one-hot encode?** Models need numbers; avoids false order between categories.
6. **What is the dummy variable trap?** Redundant column causes perfect multicollinearity; fix with `drop_first=True`.
7. **Why scale?** Prevent large-valued features dominating.
8. **Why stratify?** Preserve class proportions in the split.
9. **What is data leakage?** Test info used during training/preprocessing.
10. **Bar vs line chart?** Bar for categories; line for ordered/time data.
11. **When to use a boxplot?** Compare distributions and spot outliers across groups.
12. **What does a heatmap of correlation show?** Pairwise linear relationships; diagonal = 1.
13. **What is multicollinearity?** Input features highly correlated with each other.
14. **Why a dual-axis chart?** Two measures with different units on one time axis.
15. **What is a moving average?** Average of the last N points, sliding; smooths noise.
16. **What is seasonality?** Pattern repeating at fixed intervals.
17. **What does ARIMA(5,1,0) mean?** 5 autoregressive lags, 1 difference, 0 moving-average terms.
18. **Why difference a series?** Make it stationary (remove trend).
19. **Why `random_state` / `seed`?** Reproducibility.
20. **Why use `.get()` on dicts?** Avoid KeyError when key is missing.
21. **Why `try/except` around `read_csv(url)`?** Fall back to a local file if the network fails.
22. **Why `index=False`?** Don't write the row index as a column.
23. **What is split-apply-combine?** Group rows, apply an aggregation, merge results (`groupby`).
24. **Why `reset_index()` after groupby?** Turn group labels back into a normal column (usable in seaborn).
25. **Correlation vs causation?** Related movement doesn't prove one causes the other.

---

# MASTERY PATH (how to actually become good)

1. **Re-type** each practical from scratch without looking (muscle memory).
2. **Change the data:** new column, new chart, new groupby: the best way to learn.
3. **Break things on purpose:** remove `.copy()`, remove `+1`; read the error.
4. **Explain each chart aloud** in 3 sentences using the template in Part 11.
5. **Do 1 Kaggle dataset a week** through the full pipeline: load → clean → transform → chart → insight.
6. Next topics: histograms, KDE, pairplot, `merge`, `pivot_table`, `melt`, scikit-learn models (Logistic Regression, Random Forest), evaluation (accuracy, precision, recall, F1, confusion matrix), cross-validation.
