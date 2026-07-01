# Pandas Tutorial - Complete Documentation

## Example 1: Checking Pandas Version

```python
import pandas as pd
print(pd.__version__)
```

**Purpose:** This checks which version of pandas is installed on your system.

**Explanation:**
- `pd.__version__` is a built-in attribute that returns the version number
- Important for compatibility and debugging issues
- Different versions may have different features or syntax

**Output Example:** `1.5.3` (or whatever version you have installed)

---

## Example 2: Creating Pandas Series

```python
list1 = [10,20,30,40,50]
data = pd.Series(list1)
data = pd.Series(list1,index=["a","b","c","d","e"])
print(data)
```

**Purpose:** Introduction to Pandas Series - a one-dimensional labeled array.

**Explanation:**
- **Series:** One-dimensional array-like object containing data and labels (index)
- `pd.Series(list1)`: Creates series with default numeric index (0,1,2,3,4)
- `pd.Series(list1, index=["a","b","c","d","e"])`: Creates series with custom labels
- Series can hold any data type (integers, strings, floats, etc.)

**Output:**
```
a    10
b    20
c    30
d    40
e    50
dtype: int64
```

**Key Points:**
- Index makes data easily accessible by label
- Each element has both a position and a label

---

## Example 3: Series Indexing and Access

```python
marks = [50,60,70,80,90,100]
students = ["std1","std2","std3","std4","std5","std6"]
data = pd.Series(marks,index=students)
print(data["std6"])
```

**Purpose:** Demonstrates how to access specific elements in a Series using labels.

**Explanation:**
- Creates a Series with student names as index and marks as values
- `data["std6"]`: Accesses the value associated with label "std6"
- This is like a dictionary but with additional pandas functionality

**Output:** `100`

**Alternative Access Methods:**
```python
print(data.iloc[5])    # Access by position (6th element)
print(data.loc["std6"]) # Access by label (same as data["std6"])
```

---

## Example 4: Creating DataFrames

```python
data = {
    "Name" : ["Emp1","Emp2","Emp3","Emp4","Emp5"],
    "Age" : [25,30,35,40,45],
    "Salary" : [50000,60000,70000,80000,90000]
}
df = pd.DataFrame(data,index=["a","b","c","d","e"])
print(df)
```

**Purpose:** Introduction to DataFrames - two-dimensional labeled data structure.

**Explanation:**
- **DataFrame:** 2D structure like a table with rows and columns
- Created from dictionary where keys become column names
- `index=["a","b","c","d","e"]`: Custom row labels
- Each column is essentially a Series

**Output:**
```
   Name  Age  Salary
a  Emp1   25   50000
b  Emp2   30   60000
c  Emp3   35   70000
d  Emp4   40   80000
e  Emp5   45   90000
```

**Key Concepts:**
- Rows: Individual records/observations
- Columns: Variables/features
- Index: Row labels
- Columns: Column labels

---

## Example 6: DataFrame Operations and Analysis

```python
df = pd.read_csv("employees.csv")
print(df)
print(df.head())
print(df.tail())
print(df.shape)
print(df.columns)
print(df.info())
print(df.describe())
print(df["EmpID"])
print(df[["EmpID","Name","Salary"]])
print(df[df["Salary"]>50000])
print(df[(df["Salary"]>50000) & (df["Age"]>23)])
df["PerformanceBonus"] = df["Salary"] * 0.10
print(df.info())
df.loc[0,"Salary"] = 80000
print(df.head())
print(df.iloc[19])
print(df.shape)
df.drop("Bonus",axis=1,inplace=True)
print(df.shape)
```

**Purpose:** Comprehensive DataFrame operations including reading, exploring, filtering, and modifying data.

### Detailed Breakdown:

#### 1. Reading Data
```python
df = pd.read_csv("employees.csv")
```
- Loads data from CSV file into DataFrame
- CSV: Comma Separated Values format

#### 2. Data Exploration
```python
print(df.head())        # First 5 rows
print(df.tail())        # Last 5 rows
print(df.shape)         # (rows, columns) - e.g., (100, 5)
print(df.columns)       # Column names
print(df.info())        # Data types, memory usage, non-null counts
print(df.describe())    # Statistical summary (mean, std, min, max, etc.)
```

#### 3. Column Selection
```python
print(df["EmpID"])                    # Single column (returns Series)
print(df[["EmpID","Name","Salary"]])  # Multiple columns (returns DataFrame)
```

#### 4. Data Filtering
```python
print(df[df["Salary"]>50000])                           # Simple condition
print(df[(df["Salary"]>50000) & (df["Age"]>23)])        # Multiple conditions
```
- Use `&` for AND, `|` for OR
- Each condition must be in parentheses

#### 5. Adding New Columns
```python
df["PerformanceBonus"] = df["Salary"] * 0.10
```
- Creates new column based on existing data

#### 6. Data Modification
```python
df.loc[0,"Salary"] = 80000    # Modify specific cell by label
print(df.iloc[19])            # Access row by position
```

#### 7. Dropping Columns
```python
df.drop("Bonus", axis=1, inplace=True)
```
- `axis=1`: Drop column (axis=0 for rows)
- `inplace=True`: Modify original DataFrame

---

## Example 7: Handling Missing Data

```python
df = pd.read_csv("employees_null.csv")
print(df)
print(df.isnull())
print(df.isnull().sum())

df["Salary"] = df["Salary"].fillna(df["Salary"].mean())
print(df["Salary"])

df["City"] = df["City"].fillna("Unknown")
print(df["City"])

clean_df = df.dropna()
print(clean_df)

print(df["PerformanceRating"])
cleaned_df = df.dropna(subset=["PerformanceRating"])
print(cleaned_df["PerformanceRating"])
```

**Purpose:** Comprehensive guide to handling missing data (NaN values).

### Detailed Breakdown:

#### 1. Detecting Missing Data
```python
print(df.isnull())      # Boolean DataFrame showing True for missing values
print(df.isnull().sum()) # Count of missing values per column
```

#### 2. Filling Missing Values
```python
# Fill numerical missing values with mean
df["Salary"] = df["Salary"].fillna(df["Salary"].mean())

# Fill categorical missing values with custom value
df["City"] = df["City"].fillna("Unknown")
```

**Other fillna() options:**
- `fillna(0)`: Fill with zero
- `fillna(method='ffill')`: Forward fill (use previous value)
- `fillna(method='bfill')`: Backward fill (use next value)
- `fillna(df["Column"].median())`: Fill with median

#### 3. Dropping Missing Data
```python
clean_df = df.dropna()                              # Drop all rows with any missing values
cleaned_df = df.dropna(subset=["PerformanceRating"]) # Drop rows with missing values in specific columns
```

**dropna() parameters:**
- `axis=0`: Drop rows (default)
- `axis=1`: Drop columns
- `how='any'`: Drop if any value is missing (default)
- `how='all'`: Drop only if all values are missing
- `subset=['col1', 'col2']`: Consider only specified columns

---

## Key Pandas Concepts Summary

### 1. Data Structures
- **Series:** 1D labeled array
- **DataFrame:** 2D labeled table

### 2. Data Access
- **Label-based:** `df.loc[row, column]`
- **Position-based:** `df.iloc[row, column]`
- **Column selection:** `df["column"]` or `df[["col1", "col2"]]`

### 3. Data Filtering
- **Boolean indexing:** `df[df["column"] > value]`
- **Multiple conditions:** Use `&` (and), `|` (or)

### 4. Missing Data Strategies
- **Detection:** `isnull()`, `notnull()`
- **Removal:** `dropna()`
- **Imputation:** `fillna()`

### 5. Common Methods
- **Exploration:** `head()`, `tail()`, `info()`, `describe()`
- **Modification:** `drop()`, assignment operations
- **File I/O:** `read_csv()`, `to_csv()`

This documentation provides a solid foundation for working with pandas. Each example builds upon previous concepts, creating a comprehensive learning path from basic Series operations to advanced DataFrame manipulations and data cleaning techniques.
