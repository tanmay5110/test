# Complete Machine Learning Practicals Guide (Practicals 1 to 6)
**Format:** Full Source Code + Easy-to-Understand Technical Line-by-Line Explanation + Core Viva Concepts

---

## Practical 1: Feature Transformation — PCA on Wine Dataset

### 1. What is this Practical about? (Simple Overview)
The Wine dataset has **13 chemical measurements** (columns) for 178 wines across 3 different wine classes. It is impossible for humans to visualize a 13-dimensional graph. **Principal Component Analysis (PCA)** is a **Dimensionality Reduction** technique that compresses those 13 columns down into just **2 new columns** (**PC1** and **PC2**, called Principal Components) while keeping most of the original information (variance), allowing us to plot the data on a simple 2D X-Y scatter plot.

### 2. Full Code
```python
# --- Step 0: Imports ---
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA

# --- Step 1: Load dataset ---
url = "https://raw.githubusercontent.com/MrNobody4869/datasets/main/wine.csv"
df = pd.read_csv(url)
# df = pd.read_csv("wine.csv")
print(df.head(), "\nShape:", df.shape)

# --- Step 2: Split into features (X) and target (y) ---
y = df.iloc[:, 0]
X = df.iloc[:, 1:]

# --- Step 3: Standardise the features (PCA is scale sensitive) ---
X_scaled = StandardScaler().fit_transform(X)

# --- Step 4: PCA - full fit first to see how much variance each PC holds ---
full = PCA().fit(X_scaled)
print("\nExplained variance ratio (all PCs):\n", full.explained_variance_ratio_.round(3))
print("Cumulative:", full.explained_variance_ratio_.cumsum().round(3))

# --- Step 5: Reduce to 2 components ---
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)
print("\nVariance captured by 2 PCs: %.2f%%" % (pca.explained_variance_ratio_.sum() * 100))

# --- Step 6: Visualise ---
plt.figure(figsize=(8, 6))
for cls, c in zip(sorted(y.unique()), ["r", "g", "b"]):
    m = (y == cls).values
    plt.scatter(X_pca[m, 0], X_pca[m, 1], c=c, label=f"Class {cls}", edgecolor="k")
plt.xlabel("PC1"); plt.ylabel("PC2"); plt.title("PCA - Wine Dataset"); plt.legend()
plt.show()
```

### 3. Line-by-Line Explanation

#### Step 0: Imports
* **`import pandas as pd`**: Loads the **Pandas** library (used to read CSV files and work with data tables called DataFrames) and gives it the short nickname `pd`.
* **`import matplotlib.pyplot as plt`**: Loads the plotting module from **Matplotlib** as `plt` so we can draw 2D graphs.
* **`from sklearn.preprocessing import StandardScaler`**: Imports the `StandardScaler` tool from Scikit-Learn to bring all numerical columns onto the same scale.
* **`from sklearn.decomposition import PCA`**: Imports the `PCA` algorithm from Scikit-Learn to compress features.

#### Step 1: Load Dataset
* **`url = "https://..."`**: Stores the web link of the `wine.csv` dataset inside a string variable named `url`.
* **`df = pd.read_csv(url)`**: Reads the CSV file from the internet link and stores it as a 2D table (**DataFrame**) named `df`.
* **`print(df.head(), "\nShape:", df.shape)`**:
  * **`df.head()`**: Shows the **first 5 rows** of the table so we can verify it loaded properly.
  * **`"\nShape:"`**: `\n` moves to a new line, and prints the word `"Shape:"`.
  * **`df.shape`**: Shows `(rows, columns)` -> `(178, 14)`, meaning 178 wines and 14 columns.

#### Step 2: Split into Features ($X$) and Target ($y$)
* **`.iloc[rows, columns]`**: Stands for **Integer Location**. It selects rows and columns using index numbers starting from `0`.
* **`y = df.iloc[:, 0]`**:
  * `:` before the comma means **take all rows**.
  * `0` after the comma means **take column 0** (the first column, which contains the wine class: `1`, `2`, or `3`). We store this target label in `y`.
* **`X = df.iloc[:, 1:]`**:
  * `:` means **take all rows**.
  * `1:` means **take column 1 to the last column** (all 13 chemical measurements). We store these input features in `X`.

#### Step 3: Standardize the Features
* **`X_scaled = StandardScaler().fit_transform(X)`**:
  * **Technical Reason:** PCA looks for columns with the highest **variance** (spread of numbers). In the Wine dataset, the `Proline` column has large numbers (~`1000`) while `Hue` has tiny numbers (~`1`). Without scaling, PCA would think `Proline` is 1,000 times more important just because of its units!
  * **`StandardScaler()`**: Rescales every column using the Z-score formula $z = \frac{x - \mu}{\sigma}$ so every column has a **Mean of `0`** and a **Standard Deviation of `1`**.
  * **`.fit_transform(X)`**: Calculates the mean and standard deviation of `X` (`fit`) and immediately scales the numbers (`transform`), saving the result in `X_scaled`.

#### Step 4: Full PCA Fit (Checking Variance)
* **`full = PCA().fit(X_scaled)`**: Runs PCA keeping all 13 components so we can inspect how much information each Principal Component holds.
* **`full.explained_variance_ratio_.round(3)`**: Shows the fraction of total dataset information (variance) captured by each component (`PC1` holds ~`0.362` or 36.2%, `PC2` holds ~`0.192` or 19.2%, etc.), rounded to 3 decimal places.
* **`full.explained_variance_ratio_.cumsum().round(3)`**: **Cumulative Sum** (`.cumsum()`) keeps adding the percentages together (`PC1`, `PC1+PC2`, `PC1+PC2+PC3`...) so you can see how much total information is kept as you add more components.

#### Step 5: Reduce to 2 Components
* **`pca = PCA(n_components=2)`**: Creates a PCA object configured to keep **only the top 2 Principal Components** (`PC1` and `PC2`).
* **`X_pca = pca.fit_transform(X_scaled)`**: Compresses the `(178, 13)` scaled table down into a `(178, 2)` table stored in `X_pca`.
* **`print("\nVariance captured by 2 PCs: %.2f%%" % (pca.explained_variance_ratio_.sum() * 100))`**: Adds the variance ratios of `PC1` and `PC2` together, multiplies by `100` to get a percentage (~`55.41%`), and prints it with 2 decimal places (`%.2f%%`).

#### Step 6: Visualize the Result
* **`plt.figure(figsize=(8, 6))`**: Creates a plot window 8 inches wide by 6 inches tall.
* **`for cls, c in zip(sorted(y.unique()), ["r", "g", "b"]):`**:
  * `sorted(y.unique())` gets the unique wine classes `[1, 2, 3]`.
  * `zip(...)` pairs each class with a color: `(1, "r")` for Red, `(2, "g")` for Green, and `(3, "b")` for Blue.
* **`m = (y == cls).values`**: Creates a `True/False` filter (**Boolean mask**) that is `True` only for rows belonging to the current wine class `cls`.
* **`plt.scatter(X_pca[m, 0], X_pca[m, 1], c=c, label=f"Class {cls}", edgecolor="k")`**: Plots `PC1` (`X_pca[m, 0]`) on the X-axis and `PC2` (`X_pca[m, 1]`) on the Y-axis for that class, colors the dots (`c=c`), adds a legend label (`Class 1`), and draws a black border (`edgecolor="k"`) around each dot.
* **`plt.xlabel("PC1"); plt.ylabel("PC2"); plt.title("PCA - Wine Dataset"); plt.legend()`**: Adds axis labels, a title, and the color key box (`legend`).
* **`plt.show()`**: Displays the final graph on screen.

---

## Practical 2: Regression Analysis — Uber / Taxi Fare Prediction (Linear, Ridge, Lasso)

### 1. What is this Practical about? (Simple Overview)
We want to predict the **taxi fare amount (in dollars)** using ride details (pickup time, GPS coordinates, passenger count). Because the target is a **continuous number** (price), this is a **Regression** problem. We clean missing data, convert GPS coordinates into real trip distance in kilometers using the **Haversine formula**, remove extreme outliers using the **IQR rule**, check feature correlations, and compare three regression models: **Linear Regression**, **Ridge Regression**, and **Lasso Regression**.

### 2. Full Code
```python
# --- Step 0: Imports ---
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.metrics import r2_score, mean_squared_error

# --- Step 1 & 2: Clean data & engineer features (Datetime + Haversine Distance) ---
df = df.dropna()
df["pickup_datetime"] = pd.to_datetime(df["pickup_datetime"], errors="coerce")
df["hour"] = df["pickup_datetime"].dt.hour
df["year"] = df["pickup_datetime"].dt.year
df = df.drop(columns="pickup_datetime").dropna()

lat1, lon1, lat2, lon2 = map(np.radians, [df.pickup_latitude, df.pickup_longitude,
                                          df.dropoff_latitude, df.dropoff_longitude])
a = np.sin((lat2 - lat1) / 2) ** 2 + np.cos(lat1) * np.cos(lat2) * np.sin((lon2 - lon1) / 2) ** 2
df["distance_km"] = 6371 * 2 * np.arcsin(np.sqrt(a))

# --- Step 3: Outliers (box plot + IQR rule on fare, hard limits on the rest) ---
df.boxplot(column=["fare_amount", "distance_km"]); plt.title("Before outlier removal"); plt.show()
q1, q3 = df.fare_amount.quantile([.25, .75]); iqr = q3 - q1
mask = (df.fare_amount.between(q1 - 1.5 * iqr, q3 + 1.5 * iqr) & (df.fare_amount > 0)
        & df.passenger_count.between(1, 6) & df.distance_km.between(0.1, 100)
        & df.pickup_latitude.between(-90, 90) & df.dropoff_latitude.between(-90, 90))
print("\nRows removed as outliers:", (~mask).sum())
df = df[mask]

# --- Step 4: Correlation ---
corr = df.corr(numeric_only=True)
print("\nCorrelation with fare:\n", corr["fare_amount"].sort_values(ascending=False))
plt.imshow(corr, cmap="coolwarm"); plt.colorbar()
plt.xticks(range(len(corr)), corr.columns, rotation=90); plt.yticks(range(len(corr)), corr.columns)
plt.title("Correlation matrix"); plt.tight_layout(); plt.show()

# --- Step 5: Models (split data and define the 3 regressors) ---
X, y = df.drop(columns="fare_amount"), df["fare_amount"]
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=0)
models = {"Linear": LinearRegression(), "Ridge": Ridge(alpha=1.0), "Lasso": Lasso(alpha=0.01)}

# --- Step 6: Evaluate (train each model, score on unseen test data) ---
rows = []
for name, m in models.items():
    m.fit(X_tr, y_tr); p = m.predict(X_te)
    rows.append([name, r2_score(y_te, p), np.sqrt(mean_squared_error(y_te, p))])
print("\n", pd.DataFrame(rows, columns=["Model", "R2", "RMSE"]).round(4).to_string(index=False))
```

### 3. Line-by-Line Explanation

#### Step 1 & 2: Data Cleaning & Feature Engineering
* **`df = df.dropna()`**: Deletes any row that has empty/missing (`NaN`) cells.
* **`df["pickup_datetime"] = pd.to_datetime(df["pickup_datetime"], errors="coerce")`**: Converts raw text timestamps into Pandas `datetime` objects. `errors="coerce"` safely turns invalid/corrupted dates into `NaT` (missing value) instead of crashing the program.
* **`df["hour"] = df["pickup_datetime"].dt.hour`** & **`df["year"] = df["pickup_datetime"].dt.year`**: Extracts the hour (`0` to `23`) and year (e.g., `2014`) as numerical columns because regression math requires numbers, not date text.
* **`df = df.drop(columns="pickup_datetime").dropna()`**: Deletes the original text timestamp column and drops any rows where the date conversion failed (`NaT`).
* **`lat1, lon1, lat2, lon2 = map(np.radians, [...])`**: Converts the 4 GPS coordinate columns from degrees into **radians** (required by `np.sin` and `np.cos`).
* **`a = ...`** & **`df["distance_km"] = 6371 * 2 * np.arcsin(np.sqrt(a))`**: Applies the **Haversine Formula**, which calculates the curved-earth distance in kilometers (`6371` km is Earth's radius) between pickup and dropoff points.

#### Step 3: Outlier Detection & Removal (Box Plot + IQR)
* **`df.boxplot(column=["fare_amount", "distance_km"])`**: Draws a box plot to visually spot extreme abnormal values (dots outside the whiskers).
* **`q1, q3 = df.fare_amount.quantile([.25, .75]); iqr = q3 - q1`**:
  * `q1` is the 25th percentile (bottom 25% mark), and `q3` is the 75th percentile (top 75% mark).
  * `iqr = q3 - q1` calculates the **Interquartile Range** (the spread of the middle 50% of fares).
* **`mask = (...)`**: Creates a `True/False` filter keeping only realistic rides:
  * Fare is between `q1 - 1.5 * iqr` and `q3 + 1.5 * iqr` AND greater than `$0`.
  * `passenger_count` is between `1` and `6`.
  * `distance_km` is between `0.1` km and `100` km.
  * Latitude is valid (`-90` to `+90` degrees).
* **`print("\nRows removed as outliers:", (~mask).sum())`**: `~mask` flips `True` and `False`, so `.sum()` counts how many bad outlier rows were rejected.
* **`df = df[mask]`**: Overwrites `df` with only the clean rows.

#### Step 4: Correlation Matrix
* **`corr = df.corr(numeric_only=True)`**: Calculates the **Pearson Correlation** (`-1` to `+1`) between all numerical columns. A value near `+1` means when one column goes up, `fare_amount` strongly goes up too (like `distance_km`).
* **`plt.imshow(corr, cmap="coolwarm"); plt.colorbar()`**: Draws a color-coded heatmap of the correlations (red = positive, blue = negative).
* **`plt.xticks(...)` & `plt.yticks(...)`**: Labels the rows and columns of the heatmap with the feature names.

#### Step 5: Split Data & Define Linear, Ridge, and Lasso Models
* **`X, y = df.drop(columns="fare_amount"), df["fare_amount"]`**: `X` gets all input columns; `y` gets the target column (`fare_amount`).
* **`X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=0)`**: Splits data into **80% training (`_tr`)** and **20% testing (`_te`)**. `random_state=0` guarantees the exact same random split every time you run the script.
* **`models = {"Linear": LinearRegression(), "Ridge": Ridge(alpha=1.0), "Lasso": Lasso(alpha=0.01)}`**:
  * **Linear Regression**: Fits a straight line $\hat{y} = w_1 x_1 + w_2 x_2 + \dots + b$ with **no penalty** on weight sizes.
  * **Ridge Regression (L2 Regularization)**: Adds a squared penalty ($\alpha \sum w_i^2$) that shrinks large weights toward zero (but never exact zero) to prevent overfitting.
  * **Lasso Regression (L1 Regularization)**: Adds an absolute-value penalty ($\alpha \sum |w_i|$) that forces useless feature weights to become **exact `0`** (automatic feature selection).

#### Step 6: Train & Evaluate ($R^2$ and RMSE)
* **`m.fit(X_tr, y_tr); p = m.predict(X_te)`**: Trains each model on the training set and predicts fares `p` on the unseen test set.
* **`r2_score(y_te, p)`**: **$R^2$ Score (Coefficient of Determination)**—measures how well the model fits the data (`1.0` is a perfect fit, `0.0` is as bad as guessing the average fare).
* **`np.sqrt(mean_squared_error(y_te, p))`**: **RMSE (Root Mean Squared Error)**—tells you the average prediction error in **dollars** (lower is better).
* **`pd.DataFrame(rows, columns=["Model", "R2", "RMSE"]).round(4).to_string(index=False)`**: Prints a clean comparison table of all 3 models rounded to 4 decimal places without row index numbers.

---

## Practical 3: Classification Analysis — SVM on Handwritten Digits (0–9)

### 1. What is this Practical about? (Simple Overview)
We want a computer to look at **8×8 pixel grayscale images of handwritten digits (`0` to `9`)** and classify which digit is written. Because the output is a discrete category (`0, 1, 2, ..., 9`), this is a **Multi-Class Classification** problem. We use a **Support Vector Machine (`SVC`)**, prevent data leakage during scaling, evaluate using a **Confusion Matrix**, and compare 4 different SVM **Kernels** (`linear`, `poly`, `rbf`, `sigmoid`).

### 2. Full Code
```python
# --- Step 0: Imports ---
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score, classification_report, ConfusionMatrixDisplay

# --- Step 1: Load the dataset ---
digits = load_digits()
X, y = digits.data, digits.target
print("Images:", digits.images.shape, "| Features:", X.shape[1], "| Classes:", list(digits.target_names))

# --- Step 2: Look at a few images ---
fig, ax = plt.subplots(2, 5, figsize=(8, 4))
for a, img, lab in zip(ax.ravel(), digits.images, digits.target):
    a.imshow(img, cmap="gray_r"); a.set_title(lab); a.axis("off")
plt.show()

# --- Step 3: Split + scale ---
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42, stratify=y)
sc = StandardScaler().fit(X_tr)
X_tr, X_te = sc.transform(X_tr), sc.transform(X_te)

# --- Step 4: Train SVM (RBF kernel) ---
clf = SVC(kernel="rbf", C=10, gamma="scale").fit(X_tr, y_tr)
pred = clf.predict(X_te)

# --- Step 5: Evaluate ---
print("\nAccuracy:", round(accuracy_score(y_te, pred), 4))
print(classification_report(y_te, pred))
ConfusionMatrixDisplay.from_predictions(y_te, pred); plt.title("SVM confusion matrix"); plt.show()

# --- Step 6: Kernel comparison ---
for k in ["linear", "poly", "rbf", "sigmoid"]:
    print(f"{k:8s} accuracy: {SVC(kernel=k).fit(X_tr, y_tr).score(X_te, y_te):.4f}")

# --- Step 7: Predict a single image ---
i = 0
print("\nTest image", i, "-> predicted", clf.predict(X_te[i:i+1])[0], "| actual", y_te[i])
```

### 3. Line-by-Line Explanation

#### Step 1: Load the Dataset
* **`digits = load_digits()`**: Loads Scikit-Learn's built-in dataset of 1,797 handwritten digit photos.
* **`X, y = digits.data, digits.target`**:
  * Each image is an `8x8` grid of pixels (`digits.images`). `digits.data` (`X`) flattens each `8x8` square into a single row of **64 pixel values** (`8 * 8 = 64`).
  * `digits.target` (`y`) holds the true number (`0`–`9`) shown in each image.

#### Step 2: Visualize Sample Images
* **`fig, ax = plt.subplots(2, 5, figsize=(8, 4))`**: Creates a grid of 10 small graph boxes (2 rows × 5 columns).
* **`for a, img, lab in zip(ax.ravel(), digits.images, digits.target):`**:
  * `ax.ravel()` flattens the 2×5 grid of boxes into a 1D list of 10 boxes so we can loop over the first 10 images.
  * **`a.imshow(img, cmap="gray_r")`**: Draws the 8×8 pixel image in reverse grayscale (black ink on a white background).
  * **`a.set_title(lab); a.axis("off")`**: Puts the true digit number above the image and hides the axis rulers.

#### Step 3: Split + Leak-Free Scaling
* **`X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42, stratify=y)`**:
  * Holds out **25%** of images for testing and uses **75%** for training.
  * **`stratify=y`**: Ensures every digit (`0` through `9`) is split in the exact same proportion in both training and test sets.
* **`sc = StandardScaler().fit(X_tr)`** & **`X_tr, X_te = sc.transform(X_tr), sc.transform(X_te)`**:
  * **Technical Concept (Preventing Data Leakage):** We call `.fit(X_tr)` **only on the training data** so the scaler does not peek at the test set's average or standard deviation. Then we apply that learned scaling rule to both `X_tr` and `X_te`.

#### Step 4: Train SVM Classifier
* **`clf = SVC(kernel="rbf", C=10, gamma="scale").fit(X_tr, y_tr)`**:
  * **`SVC`**: Support Vector Classifier—finds the widest boundary (margin) separating different digit classes.
  * **`kernel="rbf"`**: Uses the **Radial Basis Function** kernel to draw smooth, curved, non-linear boundaries in high-dimensional space.
  * **`C=10`**: Penalty parameter for misclassified points. A higher `C` forces the SVM to fit training points more strictly.
  * **`gamma="scale"`**: Controls how tightly the curved boundary wraps around data clusters.
* **`pred = clf.predict(X_te)`**: Predicts the digit (`0`–`9`) for every test image.

#### Step 5: Evaluate Performance
* **`accuracy_score(y_te, pred)`**: Gives the percentage of test images predicted correctly (~`98.22%`).
* **`classification_report(y_te, pred)`**: Prints **Precision** (how accurate positive predictions were), **Recall** (what fraction of actual digits were found), and **F1-score** (balance of both) for every digit `0`–`9`.
* **`ConfusionMatrixDisplay.from_predictions(y_te, pred)`**: Draws a 10×10 grid where rows are **True digits** and columns are **Predicted digits**. Numbers on the diagonal are correct; numbers off the diagonal show which digits confused the model.

#### Step 6: Kernel Comparison
* **`for k in ["linear", "poly", "rbf", "sigmoid"]:`**: Trains 4 separate SVMs using each kernel type (`linear` = flat boundary, `poly` = polynomial curves, `rbf` = radial Gaussian curves, `sigmoid` = neural-network-style S-curve) and prints their test accuracy using `.score(X_te, y_te)`.

#### Step 7: Predict a Single Image
* **`clf.predict(X_te[i:i+1])[0]`**: Slicing `X_te[0:1]` keeps the input as a **2D array** of shape `(1, 64)` (required by Scikit-Learn's `.predict()`), and `[0]` extracts the predicted digit from the output array.

---

## Practical 4: Unsupervised Clustering — K-Means on Iris Dataset

### 1. What is this Practical about? (Simple Overview)
In **Unsupervised Learning**, we give the model **only measurements ($X$) without any labels ($y$)** and ask it to discover natural groups (**clusters**) on its own. Here, we take 150 Iris flowers with 4 measurements (Sepal Length/Width, Petal Length/Width), use the **Elbow Method** to find that the optimal number of clusters is **$k = 3$**, group the flowers using **K-Means Clustering**, and evaluate the clusters using **Silhouette Score** and **Adjusted Rand Index**.

### 2. Full Code
```python
# --- Step 1: Import the libraries ---
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import load_iris
from sklearn.metrics import silhouette_score, adjusted_rand_score

# --- Step 2: Load the dataset ---
iris = load_iris()
X, y = iris.data, iris.target
print("Data shape:", X.shape)

# --- Step 3: Elbow method - find a good number of clusters k ---
wcss = [KMeans(n_clusters=k, n_init=10, random_state=42).fit(X).inertia_ for k in range(1, 11)]
plt.plot(range(1, 11), wcss, "bo-"); plt.xlabel("k"); plt.ylabel("WCSS (inertia)")
plt.title("Elbow method"); plt.grid(True); plt.show()

# --- Step 4: Fit the final model with k = 3 (elbow bends at 3) ---
km = KMeans(n_clusters=3, n_init=10, random_state=42)
labels = km.fit_predict(X)
print("Centroids:\n", km.cluster_centers_.round(2))
print("Silhouette:", round(silhouette_score(X, labels), 3),
      "| Adjusted Rand vs species:", round(adjusted_rand_score(y, labels), 3))

# --- Step 5: Plot the clusters using petal length vs petal width ---
plt.scatter(X[:, 2], X[:, 3], c=labels, cmap="viridis", edgecolor="k")
plt.scatter(km.cluster_centers_[:, 2], km.cluster_centers_[:, 3], c="red", s=200, marker="X", label="Centroids")
plt.xlabel("Petal length"); plt.ylabel("Petal width"); plt.title("K-Means (k=3) on Iris"); plt.legend(); plt.show()
```

### 3. Line-by-Line Explanation

#### Step 1 & 2: Imports & Load Dataset
* **`iris = load_iris()`** & **`X, y = iris.data, iris.target`**:
  * `X` has shape `(150, 4)`: 150 flowers, 4 measurements (`Sepal Length`, `Sepal Width`, `Petal Length`, `Petal Width`).
  * `y` has the true flower species (`0, 1, 2`). **Important:** We **do not** pass `y` to `KMeans` during training! We only use `y` at the end to verify how well K-Means grouped them.

#### Step 3: The Elbow Method (Finding $k$)
* **`wcss = [KMeans(n_clusters=k, n_init=10, random_state=42).fit(X).inertia_ for k in range(1, 11)]`**:
  * Runs K-Means for `k = 1, 2, ..., 10` clusters.
  * **`n_init=10`**: Runs the algorithm 10 times with different random starting center points and picks the best run.
  * **`.inertia_`**: Returns **WCSS (Within-Cluster Sum of Squares)**—the total squared distance from every point to its cluster center (**Centroid**). Smaller WCSS means tighter clusters.
* **`plt.plot(range(1, 11), wcss, "bo-")`**: Plots `k` vs `WCSS` using blue circles connected by a solid line (`"bo-"`). The curve bends sharply like an arm's elbow at **`k = 3`**, proving 3 clusters is ideal.

#### Step 4: Fit Final K-Means Model ($k=3$) & Score It
* **`km = KMeans(n_clusters=3, n_init=10, random_state=42)`**: Creates the K-Means model with 3 clusters.
* **`labels = km.fit_predict(X)`**: Finds the 3 clusters and assigns every flower a cluster group number (`0`, `1`, or `2`). Notice only `X` is passed!
* **`km.cluster_centers_.round(2)`**: Prints the `[Sepal L, Sepal W, Petal L, Petal W]` coordinates of the **3 Centroids** (the center point of each cluster).
* **`silhouette_score(X, labels)`**: Measures how compact and well-separated the clusters are **without knowing true species labels** (ranges from `-1` to `+1`; higher is better, ~`0.553` here).
* **`adjusted_rand_score(y, labels)`**: Compares the K-Means clusters (`labels`) against the true botanical species (`y`), scoring from `0.0` (random) to `1.0` (perfect match, ~`0.73` here).

#### Step 5: Visualize Clusters & Centroids
* **`plt.scatter(X[:, 2], X[:, 3], c=labels, cmap="viridis", edgecolor="k")`**: Plots **Petal Length** (column `2`) vs **Petal Width** (column `3`) and colors each flower dot according to its K-Means cluster (`c=labels`).
* **`plt.scatter(km.cluster_centers_[:, 2], km.cluster_centers_[:, 3], c="red", s=200, marker="X", label="Centroids")`**: Plots the 3 cluster centers on top as big (`s=200`) red `"X"` markers.

---

## Practical 5: Ensemble Learning — Random Forest on Car Evaluation Dataset

### 1. What is this Practical about? (Simple Overview)
We want to predict whether a car is **Unacceptable (`unacc`), Acceptable (`acc`), Good (`good`), or Very Good (`vgood`)** based on 6 text attributes (buying price, maintenance cost, doors, passenger capacity, luggage boot size, and safety). Since all inputs are ordered text words (`"low"`, `"med"`, `"high"`), we first convert them into ordered numbers (`0, 1, 2`) using **`OrdinalEncoder`**, train a **Random Forest Classifier** (an ensemble of 100 Decision Trees voting together), and plot **Feature Importance** to see which car attributes matter most.

### 2. Full Code
```python
# --- Step 1: Import the libraries ---
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import OrdinalEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report, ConfusionMatrixDisplay

# --- Step 2: Load the dataset (file has no header row) ---
cols = ["buying", "maint", "doors", "persons", "lug_boot", "safety", "class"]
url = "https://raw.githubusercontent.com/MrNobody4869/datasets/main/car_evaluation.csv"
df = pd.read_csv(url, header=None, names=cols)
print(df.head(), "\nShape:", df.shape, "\n\nClass counts:\n", df["class"].value_counts())

# --- Step 3: Encode text categories as numbers (ordered categories keep their natural order) ---
order = [["low", "med", "high", "vhigh"], ["low", "med", "high", "vhigh"], ["2", "3", "4", "5more"],
         ["2", "4", "more"], ["small", "med", "big"], ["low", "med", "high"]]
X = pd.DataFrame(OrdinalEncoder(categories=order).fit_transform(df[cols[:-1]]), columns=cols[:-1])
y = df["class"]

# --- Step 4: Split into training and testing data ---
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

# --- Step 5: Train the Random Forest ---
rf = RandomForestClassifier(n_estimators=100, random_state=42).fit(X_tr, y_tr)
pred = rf.predict(X_te)

# --- Step 6: Evaluate on the unseen test data ---
print("\nAccuracy:", round(accuracy_score(y_te, pred), 4))
print(classification_report(y_te, pred))
ConfusionMatrixDisplay.from_predictions(y_te, pred, xticks_rotation=45); plt.tight_layout(); plt.show()

# --- Step 7: Feature importance (which inputs mattered most) ---
imp = pd.Series(rf.feature_importances_, index=X.columns).sort_values()
print("\nFeature importance:\n", imp.sort_values(ascending=False).round(3))
imp.plot.barh(title="Random Forest feature importance"); plt.show()
```

### 3. Line-by-Line Explanation

#### Step 2: Load Headerless Dataset
* **`cols = ["buying", "maint", "doors", "persons", "lug_boot", "safety", "class"]`**: Creates a list of column names because the raw CSV file has no title row at the top.
* **`df = pd.read_csv(url, header=None, names=cols)`**: `header=None` tells Pandas the first line of the file is data (not headers), and `names=cols` attaches our custom column names.
* **`df["class"].value_counts()`**: Counts how many cars are in each category (`unacc`: 1210, `acc`: 384, `good`: 69, `vgood`: 65), showing that the classes are **imbalanced**.

#### Step 3: Ordinal Encoding (Text to Ranked Numbers)
* **`order = [...]`**: Lists the natural smallest-to-largest order for each of the 6 input columns so `"low"` becomes `0`, `"med"` becomes `1`, `"high"` becomes `2`, and `"vhigh"` becomes `3`. (Without `categories=order`, Python would sort alphabetically and put `"high"` before `"low"`!).
* **`X = pd.DataFrame(OrdinalEncoder(categories=order).fit_transform(df[cols[:-1]]), columns=cols[:-1])`**:
  * `cols[:-1]` selects the first 6 columns (all except `"class"`).
  * `OrdinalEncoder(...).fit_transform(...)` replaces the words with ordered numbers `0, 1, 2, 3`.
  * `pd.DataFrame(..., columns=cols[:-1])` keeps the column names attached for plotting in Step 7.
* **`y = df["class"]`**: Stores the target car rating in `y`.

#### Step 4 & 5: Split Data & Train Random Forest
* **`train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)`**: Keeps 80% for training and 20% for testing while preserving exact class ratios (`stratify=y`).
* **`rf = RandomForestClassifier(n_estimators=100, random_state=42).fit(X_tr, y_tr)`**:
  * **How Random Forest Works:** Instead of relying on 1 Decision Tree (which easily overfits), it trains **100 different Decision Trees (`n_estimators=100`)** on random subsets of data and features. When predicting a test car (`rf.predict(X_te)`), all 100 trees vote and the majority wins.

#### Step 6 & 7: Evaluate & Plot Feature Importance
* **`classification_report(y_te, pred)`** & **`ConfusionMatrixDisplay...`**: Evaluates accuracy (~`97.7%`) and checks Precision/Recall for rare classes like `good` and `vgood`.
* **`imp = pd.Series(rf.feature_importances_, index=X.columns).sort_values()`**:
  * `rf.feature_importances_` gives a score for each of the 6 features (summing to `1.0`) showing how heavily the 100 trees relied on that column.
  * **`imp.plot.barh(...)`**: Draws a horizontal bar chart showing that **`safety`** (~30%) and **`persons`** (~23%) are the most important factors in rating a car.

---

## Practical 6: Reinforcement Learning — Q-Learning on Tic-Tac-Toe

### 1. What is this Practical about? (Simple Overview)
In **Reinforcement Learning (RL)**, there is no dataset! Instead, an **Agent** learns by interacting with an **Environment** (the Tic-Tac-Toe board) through trial and error, receiving **Rewards** (`+1` for winning, `+0.5` for a draw, `-1` for losing). Using **Q-Learning**, our agent plays **30,000 games against itself (Self-Play)**, stores the value of every `(board_state, move)` inside a lookup dictionary called a **Q-Table**, and learns an unbeatable strategy.

### 2. Full Code
```python
# --- Step 1: Import the libraries ---
import random, sys
from collections import defaultdict
import matplotlib.pyplot as plt

# --- Step 2: Environment - the board and the rules of the game ---
LINES = [(0,1,2),(3,4,5),(6,7,8),(0,3,6),(1,4,7),(2,5,8),(0,4,8),(2,4,6)]

def winner(b):
    for i, j, k in LINES:
        if b[i] and b[i] == b[j] == b[k]:
            return b[i]
    return 0 if 0 in b else -1

def moves(b):       return [i for i in range(9) if b[i] == 0]
def play(b, a, p):  return b[:a] + (p,) + b[a+1:]
def flip(b):        return tuple((0, 2, 1)[c] for c in b)

# --- Step 3: Q-learning agent - the Q-table, epsilon-greedy choice ---
Q = defaultdict(lambda: [0.0] * 9)
ALPHA, GAMMA = 0.3, 0.95

def best_q(s):  return max(Q[s][m] for m in moves(s))

def choose(s, eps):
    mv = moves(s)
    if random.random() < eps:
        return random.choice(mv)
    top = best_q(s)
    return random.choice([m for m in mv if Q[s][m] == top])

# --- Step 4: Training by self-play (one Q-table plays both sides) ---
def train(episodes=30000, eps_start=1.0, eps_end=0.05):
    for ep in range(episodes):
        eps = max(eps_end, eps_start * (1 - ep / (episodes * 0.8)))
        b, player, hist = (0,) * 9, 1, []
        while True:
            s = b if player == 1 else flip(b)
            a = choose(s, eps)
            hist.append((s, a))
            b = play(b, a, player)
            w = winner(b)
            if w != 0:
                break
            player = 3 - player
        won = w != -1
        for i in range(len(hist) - 1, -1, -1):
            s_i, a_i = hist[i]
            if i == len(hist) - 1:
                target = 1.0 if won else 0.5
            elif i == len(hist) - 2:
                target = -1.0 if won else 0.5
            else:
                target = -GAMMA * best_q(hist[i + 1][0])
            Q[s_i][a_i] += ALPHA * (target - Q[s_i][a_i])

# --- Step 5: Testing - agent against a random player ---
def agent_vs_random(games=1000):
    res = {"win": 0, "draw": 0, "loss": 0}
    for g in range(games):
        ag = 1 if g % 2 == 0 else 2
        b, p = (0,) * 9, 1
        while True:
            a = choose(b if p == 1 else flip(b), 0) if p == ag else random.choice(moves(b))
            b = play(b, a, p); w = winner(b)
            if w != 0:
                res["draw" if w == -1 else "win" if w == ag else "loss"] += 1
                break
            p = 3 - p
    return res

def show(b):
    sym = {0: ".", 1: "X", 2: "O"}
    for r in range(3):
        print(" ".join(sym[b[3 * r + c]] for c in range(3)))

def human_game():
    b, p = (0,) * 9, 1
    while True:
        show(b)
        if p == 1:
            try:
                a = int(input("Your move (cell 0-8): "))
            except ValueError:
                continue
            if a not in moves(b):
                print("Invalid move"); continue
        else:
            a = choose(flip(b), 0)
        b = play(b, a, p); w = winner(b)
        if w != 0:
            show(b); print("Draw" if w == -1 else "You win" if w == 1 else "Agent wins"); return
        p = 3 - p

# --- Step 6: Run everything ---
if __name__ == "__main__":
    random.seed(0)
    print("Untrained agent vs random:", agent_vs_random(500))
    train()
    print("States learned:", len(Q))
    r = agent_vs_random(1000)
    print("Trained agent vs random  :", r)
    plt.bar(list(r.keys()), list(r.values()), color=["g", "gray", "r"])
    plt.title("Q-agent vs random player (1000 games)"); plt.show()
    if "--play" in sys.argv:
        human_game()
```

### 3. Line-by-Line Explanation

#### Step 2: The Board & Environment Functions
* **Board Representation**: A 9-number tuple `(0, 0, 0, 0, 0, 0, 0, 0, 0)` representing cells `0` to `8` (`0` = empty, `1` = X, `2` = O). We use a `tuple` instead of a `list` because dictionary keys in Python (`Q[state]`) must be immutable (un-editable).
* **`LINES`**: All 8 winning 3-in-a-row combinations (3 rows, 3 columns, 2 diagonals).
* **`winner(b)`**: Checks all 8 `LINES`. If any line has three matching `1`s or `2`s, returns that winner (`1` or `2`). Otherwise returns `0` if empty cells remain, or `-1` for a Draw.
* **`moves(b)`**: Returns a list of empty cell indices (`0`–`8`) where `b[i] == 0`.
* **`play(b, a, p)`**: Returns a new board tuple with player `p` placed at cell `a`.
* **`flip(b)`**: Swaps `1` and `2` on the board so that whoever's turn it is always sees their own pieces as `1`. This allows **one single Q-table** to learn both X and O!

#### Step 3: Q-Table & Epsilon-Greedy Action Selection
* **`Q = defaultdict(lambda: [0.0] * 9)`**: Our **Q-Table**. For any board state `s`, `Q[s]` holds 9 decimal numbers representing the score (expected reward) of playing in cells `0` through `8`. Unseen board states automatically start at `[0.0, ..., 0.0]`.
* **`ALPHA = 0.3` (Learning Rate)**: How strongly new game results update old Q-values (30% new, 70% old).
* **`GAMMA = 0.95` (Discount Factor)**: How much future moves' rewards carry back to earlier moves.
* **`best_q(s)`**: Returns the highest Q-value among all currently legal empty squares on board `s`.
* **`choose(s, eps)` (Epsilon-Greedy Strategy)**:
  * If a random number (`0.0` to `1.0`) is less than `eps` ($\epsilon$), the agent **Explores** by picking a random legal move.
  * Otherwise, the agent **Exploits** its knowledge by picking the move with the highest Q-value (`best_q(s)`), breaking ties randomly.

#### Step 4: Self-Play Training & Backward Bellman Update
* **`eps = max(eps_end, eps_start * (1 - ep / (episodes * 0.8)))`**: Gradually reduces `eps` from `1.0` (100% random guessing at the start) down to `0.05` (95% smart play) so the agent explores early on and refines its best moves later.
* **`while True:` loop**: Plays one complete game of Tic-Tac-Toe, saving every `(state, action)` pair in `hist`, and flipping the turn using `player = 3 - player` (`3-1=2`, `3-2=1`).
* **`for i in range(len(hist) - 1, -1, -1):` (Bellman Update from End to Start)**:
  * Loops **backward** from the last move of the game to the first move:
  * **Final move (`len(hist) - 1`)**: `target = 1.0` if that move won the game, or `0.5` if it drew.
  * **Second-to-last move (`len(hist) - 2`, the losing player's last move)**: `target = -1.0` if the opponent won on the next turn, or `0.5` if it drew.
  * **All earlier moves (`else`)**: `target = -GAMMA * best_q(hist[i + 1][0])`. The minus sign (`-`) is there because Tic-Tac-Toe is a **zero-sum game**: a high score for your opponent on the next turn is a bad score for you!
  * **`Q[s_i][a_i] += ALPHA * (target - Q[s_i][a_i])`**: Updates the Q-table entry toward `target`.

#### Step 5 & 6: Evaluation vs. Random Player & Human Mode
* **`agent_vs_random(games=1000)`**: Plays 1,000 test games against a purely random opponent with `eps=0` (100% best moves, no random exploration) and counts wins, draws, and losses.
* **Before vs. After Training**:
  * **Untrained Agent:** Wins ~45%, Loses ~45%.
  * **Trained Agent (after 30,000 self-play games):** Learns ~4,500+ board states, wins **~85–90%+** of games, and reduces losses to almost **0**!
* **`human_game()`**: Lets you play interactively against the trained AI in the terminal if you run `python script.py --play`.

---

## Quick Viva / Exam Cheat Sheet (All 6 Practicals)

| Practical | Algorithm Used | Learning Type | Why Scaling / Encoding? | Evaluation Metrics Used |
| :--- | :--- | :--- | :--- | :--- |
| **1. Wine** | **PCA** (Principal Component Analysis) | Unsupervised (Feature Transformation) | **`StandardScaler`**: PCA is scale-sensitive; prevents large-unit columns from dominating variance. | **Explained Variance Ratio** & **Cumulative Variance** |
| **2. Taxi Fare** | **Linear, Ridge (L2), Lasso (L1)** | Supervised (**Regression**) | **Haversine Formula** (GPS to km) + **IQR Rule** (Outlier removal) | **$R^2$ Score** (closer to 1 is better) & **RMSE** (error in $) |
| **3. Digits (0–9)** | **SVM (`SVC`)** with Kernels | Supervised (**Classification**) | **`StandardScaler` fit on `X_tr` only** to prevent Data Leakage. | **Accuracy**, **Precision, Recall, F1**, **Confusion Matrix** |
| **4. Iris** | **K-Means Clustering** ($k=3$) | **Unsupervised** (**Clustering**) | **Elbow Method (WCSS / Inertia)** used to find optimal $k=3$. | **Silhouette Score** (no labels needed) & **Adjusted Rand Index** |
| **5. Car Eval** | **Random Forest** (100 Trees) | Supervised (**Ensemble Classification**) | **`OrdinalEncoder`**: Preserves natural rank (`low < med < high`). | **Accuracy**, **Classification Report**, **Feature Importance** |
| **6. Tic-Tac-Toe** | **Q-Learning** (Self-Play) | **Reinforcement Learning** | **Epsilon-Greedy ($\epsilon$)** balances Exploration vs. Exploitation. | **Win / Draw / Loss Rate** vs. Random Opponent |