# Complete Pandas Functions Reference Guide with Detailed Explanations

## 📊 **DataFrame Creation & Basic Info**

### **DataFrame Creation**

```python
# Creating DataFrames from various data sources

# 1. Basic DataFrame creation
pd.DataFrame(data, index=None, columns=None, dtype=None)
```
**Explanation:** Creates a DataFrame from various data types (lists, dictionaries, arrays, etc.)
- `data`: Input data (list, dict, numpy array, etc.)
- `index`: Row labels (if None, uses default 0,1,2...)
- `columns`: Column labels (if None, uses default or infers from data)
- `dtype`: Data type to force (optional)

**Example:**
```python
import pandas as pd
import numpy as np

# From dictionary
data = {'Name': ['Alice', 'Bob', 'Charlie'], 'Age': [25, 30, 35]}
df = pd.DataFrame(data)
print(df)
#      Name  Age
# 0   Alice   25
# 1     Bob   30
# 2 Charlie   35
```

```python
# 2. Create DataFrame from dictionary with specific orientation
pd.DataFrame.from_dict(data, orient='columns', dtype=None)
```
**Explanation:** Creates DataFrame from dictionary with control over orientation
- `orient='columns'`: Keys become column names (default)
- `orient='index'`: Keys become row indices
- `orient='tight'`: Assumes keys are ['index', 'columns', 'data']

**Example:**
```python
# Column orientation (default)
data = {'A': [1, 2, 3], 'B': [4, 5, 6]}
df1 = pd.DataFrame.from_dict(data, orient='columns')

# Index orientation
data = {'row1': [1, 2], 'row2': [3, 4]}
df2 = pd.DataFrame.from_dict(data, orient='index', columns=['A', 'B'])
```

```python
# 3. Create DataFrame from records (list of tuples/lists)
pd.DataFrame.from_records(data, index=None, exclude=None)
```
**Explanation:** Creates DataFrame from structured data like list of tuples
- `data`: List of tuples, lists, or structured arrays
- `index`: Column to use as index
- `exclude`: Columns to exclude

**Example:**
```python
records = [('Alice', 25, 'Engineer'), ('Bob', 30, 'Doctor')]
df = pd.DataFrame.from_records(records, columns=['Name', 'Age', 'Job'])
```

### **Reading Data from Files**

```python
# 4. Read CSV files
pd.read_csv(filepath, sep=',', header='infer', names=None, index_col=None)
```
**Explanation:** Reads comma-separated values file into DataFrame
- `filepath`: Path to the CSV file
- `sep`: Delimiter (comma by default)
- `header`: Row number to use as column names ('infer' auto-detects)
- `names`: List of column names to use
- `index_col`: Column to use as row labels

**Example:**
```python
# Basic CSV reading
df = pd.read_csv('data.csv')

# Custom separator and no header
df = pd.read_csv('data.txt', sep='\t', header=None, names=['Col1', 'Col2'])

# Use first column as index
df = pd.read_csv('data.csv', index_col=0)
```

```python
# 5. Read Excel files
pd.read_excel(io, sheet_name=0, header=0, names=None, index_col=None)
```
**Explanation:** Reads Excel file into DataFrame
- `io`: Path to Excel file
- `sheet_name`: Sheet to read (0 for first sheet, name for specific sheet)
- `header`: Row to use for column names
- `names`: List of column names
- `index_col`: Column to use as index

**Example:**
```python
# Read first sheet
df = pd.read_excel('data.xlsx')

# Read specific sheet
df = pd.read_excel('data.xlsx', sheet_name='Sales')

# Read multiple sheets
dfs = pd.read_excel('data.xlsx', sheet_name=['Sheet1', 'Sheet2'])
```

```python
# 6. Read JSON files
pd.read_json(path_or_buf, orient=None, typ='frame', lines=False)
```
**Explanation:** Reads JSON data into DataFrame
- `path_or_buf`: JSON file path or JSON string
- `orient`: Expected JSON format ('records', 'index', 'values', etc.)
- `typ`: Type of object to recover ('frame' for DataFrame, 'series' for Series)
- `lines`: Read file as one JSON object per line

**Example:**
```python
# Read JSON file
df = pd.read_json('data.json')

# Read JSON lines format
df = pd.read_json('data.jsonl', lines=True)
```

### **Basic DataFrame Information**

```python
# 7. Get DataFrame dimensions
df.shape
```
**Explanation:** Returns tuple of (number of rows, number of columns)
**Example:**
```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
print(df.shape)  # Output: (3, 2) - 3 rows, 2 columns
```

```python
# 8. Get total number of elements
df.size
```
**Explanation:** Returns total number of elements (rows × columns)
**Example:**
```python
print(df.size)  # Output: 6 (3 rows × 2 columns)
```

```python
# 9. Get number of dimensions
df.ndim
```
**Explanation:** Returns number of dimensions (always 2 for DataFrame)
**Example:**
```python
print(df.ndim)  # Output: 2
```

```python
# 10. Get comprehensive information about DataFrame
df.info(verbose=None, buf=None, max_cols=None, memory_usage=None)
```
**Explanation:** Prints concise summary of DataFrame including data types, non-null counts, memory usage
- `verbose`: Whether to print full summary
- `buf`: Buffer to write to
- `max_cols`: Maximum columns to display
- `memory_usage`: Whether to display memory usage

**Example:**
```python
df.info()
# Output:
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 3 entries, 0 to 2
# Data columns (total 2 columns):
#  #   Column  Non-Null Count  Dtype
# ---  ------  --------------  -----
#  0   A       3 non-null      int64
#  1   B       3 non-null      int64
```

```python
# 11. Generate descriptive statistics
df.describe(percentiles=None, include=None, exclude=None)
```
**Explanation:** Generates descriptive statistics (count, mean, std, min, quartiles, max)
- `percentiles`: Percentiles to include (default: [.25, .5, .75])
- `include`: Data types to include ('all', 'number', 'object', etc.)
- `exclude`: Data types to exclude

**Example:**
```python
df = pd.DataFrame({'A': [1, 2, 3, 4, 5], 'B': [10, 20, 30, 40, 50]})
print(df.describe())
#               A          B
# count  5.000000   5.000000
# mean   3.000000  30.000000
# std    1.581139  15.811388
# min    1.000000  10.000000
# 25%    2.000000  20.000000
# 50%    3.000000  30.000000
# 75%    4.000000  40.000000
# max    5.000000  50.000000
```

---

## 🔍 **Data Selection & Indexing**

### **Column Selection**

```python
# 12. Select single column (returns Series)
df['column_name']
```
**Explanation:** Selects one column and returns it as a pandas Series
**Example:**
```python
df = pd.DataFrame({'Name': ['Alice', 'Bob'], 'Age': [25, 30]})
ages = df['Age']  # Returns Series
print(type(ages))  # <class 'pandas.core.series.Series'>
```

```python
# 13. Select multiple columns (returns DataFrame)
df[['col1', 'col2']]
```
**Explanation:** Selects multiple columns using list of column names, returns DataFrame
**Example:**
```python
subset = df[['Name', 'Age']]  # Returns DataFrame with selected columns
print(type(subset))  # <class 'pandas.core.frame.DataFrame'>
```

```python
# 14. Attribute access (dot notation)
df.column_name
```
**Explanation:** Access column using dot notation (only works if column name is valid Python identifier)
**Example:**
```python
ages = df.Age  # Same as df['Age'], but only works if 'Age' is valid identifier
# Won't work for: df.First Name (space), df.2021 (starts with number)
```

```python
# 15. Select columns by data type
df.select_dtypes(include=None, exclude=None)
```
**Explanation:** Selects columns based on their data types
- `include`: Data types to include (e.g., 'number', 'object', 'datetime')
- `exclude`: Data types to exclude

**Example:**
```python
df = pd.DataFrame({
    'Name': ['Alice', 'Bob'],
    'Age': [25, 30],
    'Salary': [50000.0, 60000.0],
    'Date': pd.to_datetime(['2021-01-01', '2021-01-02'])
})

# Select only numeric columns
numeric_cols = df.select_dtypes(include=['number'])
# Select only object (string) columns
string_cols = df.select_dtypes(include=['object'])
# Exclude datetime columns
no_datetime = df.select_dtypes(exclude=['datetime'])
```

### **Row Selection**

```python
# 16. Label-based indexing with loc
df.loc[row_indexer, column_indexer]
```
**Explanation:** Selects rows and columns by labels/names
- `row_indexer`: Row labels, boolean array, or slice
- `column_indexer`: Column labels or slice (optional)

**Example:**
```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]}, index=['x', 'y', 'z'])

# Select single row
row = df.loc['x']  # Returns Series

# Select multiple rows
rows = df.loc['x':'y']  # Returns DataFrame

# Select specific rows and columns
subset = df.loc['x':'y', 'A':'B']

# Boolean indexing
filtered = df.loc[df['A'] > 1]
```

```python
# 17. Position-based indexing with iloc
df.iloc[row_indexer, column_indexer]
```
**Explanation:** Selects rows and columns by integer positions (0-based)
- `row_indexer`: Integer positions, slice, or list of integers
- `column_indexer`: Integer positions or slice (optional)

**Example:**
```python
# Select first row
first_row = df.iloc[0]

# Select first 2 rows
first_two = df.iloc[0:2]

# Select specific positions
subset = df.iloc[0:2, 0:1]  # First 2 rows, first column

# Select last row
last_row = df.iloc[-1]
```

```python
# 18. Boolean indexing
df[df['column'] > value]
```
**Explanation:** Filters rows based on boolean conditions
**Example:**
```python
df = pd.DataFrame({'Name': ['Alice', 'Bob', 'Charlie'], 'Age': [25, 30, 35]})

# Filter rows where Age > 25
adults = df[df['Age'] > 25]

# Multiple conditions (use & for AND, | for OR)
filtered = df[(df['Age'] > 25) & (df['Age'] < 35)]

# Check if values are in a list
subset = df[df['Name'].isin(['Alice', 'Bob'])]
```

```python
# 19. Query method for filtering
df.query('column > value')
```
**Explanation:** Filters DataFrame using query string (more readable for complex conditions)
**Example:**
```python
# Simple condition
result = df.query('Age > 25')

# Multiple conditions
result = df.query('Age > 25 and Age < 35')

# Using variables
min_age = 25
result = df.query('Age > @min_age')
```

---

## 🧹 **Data Cleaning & Preprocessing**

### **Missing Data Handling**

```python
# 20. Check for missing values
df.isna()  # or df.isnull()
```
**Explanation:** Returns DataFrame of same shape with True where values are missing (NaN, None, etc.)
**Example:**
```python
df = pd.DataFrame({'A': [1, 2, None], 'B': [4, None, 6]})
print(df.isna())
#       A      B
# 0  False  False
# 1  False   True
# 2   True  False

# Check if any column has missing values
print(df.isna().any())
# A    True
# B    True

# Count missing values per column
print(df.isna().sum())
# A    1
# B    1
```

```python
# 21. Remove rows/columns with missing values
df.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
```
**Explanation:** Removes rows or columns containing missing values
- `axis`: 0 for rows, 1 for columns
- `how`: 'any' (if any NA values), 'all' (if all values are NA)
- `thresh`: Minimum number of non-NA values required
- `subset`: Labels to consider for missing values
- `inplace`: Modify original DataFrame if True

**Example:**
```python
df = pd.DataFrame({'A': [1, 2, None], 'B': [4, None, 6], 'C': [7, 8, 9]})

# Drop rows with any missing values
clean_df = df.dropna()  # Keeps only rows with no missing values

# Drop rows with missing values in specific columns
clean_df = df.dropna(subset=['A'])

# Drop rows with less than 2 non-missing values
clean_df = df.dropna(thresh=2)

# Drop columns with any missing values
clean_df = df.dropna(axis=1)
```

```python
# 22. Fill missing values
df.fillna(value=None, method=None, axis=None, inplace=False, limit=None)
```
**Explanation:** Fills missing values with specified value or method
- `value`: Value to use for filling (scalar, dict, Series, or DataFrame)
- `method`: 'ffill'/'pad' (forward fill), 'bfill'/'backfill' (backward fill)
- `limit`: Maximum number of consecutive NAs to fill

**Example:**
```python
df = pd.DataFrame({'A': [1, None, 3], 'B': [None, 2, 3]})

# Fill with specific value
filled = df.fillna(0)

# Fill with different values per column
filled = df.fillna({'A': 0, 'B': 999})

# Forward fill (use previous value)
filled = df.fillna(method='ffill')

# Fill with column mean
filled = df.fillna(df.mean())
```

### **Data Type Conversion**

```python
# 23. Convert data types
df.astype(dtype, copy=True, errors='raise')
```
**Explanation:** Converts DataFrame or Series to specified data type
- `dtype`: Target data type (str, int, float, etc.) or dict for multiple columns
- `copy`: Return copy if True
- `errors`: 'raise' (default), 'ignore' (invalid parsing returns original)

**Example:**
```python
df = pd.DataFrame({'A': ['1', '2', '3'], 'B': [1.1, 2.2, 3.3]})

#
