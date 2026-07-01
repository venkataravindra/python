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


### **1. Sorting Functions**

#### **`df.sort_values()`**
```python
# Basic sorting
df.sort_values('column_name')
```
**Purpose:** Sorts DataFrame by values in specified column(s)
**Parameters:**
- `by`: Column name(s) to sort by
- `ascending`: True (default) for ascending, False for descending
- `inplace`: If True, modifies original DataFrame
- `na_position`: Where to put NaN values ('last' or 'first')

**Example:**
```python
import pandas as pd
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 20],
    'salary': [50000, 60000, 45000]
})

# Sort by age (ascending)
sorted_df = df.sort_values('age')
print(sorted_df)
# Output:
#      name  age  salary
# 2  Charlie   20   45000
# 0    Alice   25   50000
# 1      Bob   30   60000
```

#### **Multiple Column Sorting**
```python
df.sort_values(['col1', 'col2'], ascending=[True, False])
```
**Explanation:** 
- Sorts by `col1` first (ascending), then by `col2` (descending)
- `ascending` list must match number of columns

**Example:**
```python
df = pd.DataFrame({
    'department': ['IT', 'HR', 'IT', 'HR'],
    'salary': [70000, 60000, 80000, 65000],
    'age': [25, 30, 35, 28]
})

# Sort by department (ascending), then salary (descending)
result = df.sort_values(['department', 'salary'], ascending=[True, False])
print(result)
# Output: HR department first, then IT, with higher salaries first within each department
```

#### **`df.sort_index()`**
```python
df.sort_index()
```
**Purpose:** Sorts DataFrame by index labels
**When to use:** When your index is meaningful (dates, categories) and you want to order by it

**Example:**
```python
df = pd.DataFrame({
    'value': [10, 20, 30]
}, index=['c', 'a', 'b'])

sorted_df = df.sort_index()
print(sorted_df)
# Output:
#    value
# a     20
# b     30
# c     10
```

### **2. Ranking Functions**

#### **`df.rank()`**
```python
df['column'].rank()
```
**Purpose:** Assigns ranks to values (1 = smallest value by default)
**Parameters:**
- `method`: How to handle ties ('average', 'min', 'max', 'first', 'dense')
- `ascending`: True for 1=smallest, False for 1=largest

**Example:**
```python
df = pd.DataFrame({
    'score': [85, 92, 78, 92, 88]
})

df['rank'] = df['score'].rank()
df['rank_dense'] = df['score'].rank(method='dense')
print(df)
# Output:
#    score  rank  rank_dense
# 0     85   2.0         2.0
# 1     92   4.5         3.0  # Tied values get average rank
# 2     78   1.0         1.0
# 3     92   4.5         3.0  # Same as above
# 4     88   3.0         2.0
```

**Rank Methods Explained:**
- `'average'`: Average of ranks for tied values (default)
- `'min'`: Minimum rank for tied values
- `'max'`: Maximum rank for tied values
- `'first'`: Ranks assigned in order they appear
- `'dense'`: Like 'min' but no gaps in ranking

### **3. Unique Values and Counts**

#### **`df['column'].unique()`**
```python
df['column'].unique()
```
**Purpose:** Returns array of unique values in a column
**Returns:** NumPy array of unique values

**Example:**
```python
df = pd.DataFrame({
    'city': ['NYC', 'LA', 'NYC', 'Chicago', 'LA']
})

unique_cities = df['city'].unique()
print(unique_cities)
# Output: ['NYC' 'LA' 'Chicago']
print(type(unique_cities))  # <class 'numpy.ndarray'>
```

#### **`df['column'].nunique()`**
```python
df['column'].nunique()
```
**Purpose:** Returns count of unique values
**Parameters:**
- `dropna`: If True (default), excludes NaN from count

**Example:**
```python
df = pd.DataFrame({
    'city': ['NYC', 'LA', 'NYC', None, 'LA']
})

print(df['city'].nunique())         # Output: 2 (excludes NaN)
print(df['city'].nunique(dropna=False))  # Output: 3 (includes NaN)
```

#### **`df['column'].value_counts()`**
```python
df['column'].value_counts()
```
**Purpose:** Returns count of each unique value, sorted by frequency
**Parameters:**
- `normalize`: If True, returns proportions instead of counts
- `sort`: If True (default), sorts by frequency
- `ascending`: If True, sorts ascending
- `dropna`: If True (default), excludes NaN

**Example:**
```python
df = pd.DataFrame({
    'grade': ['A', 'B', 'A', 'C', 'B', 'A']
})

counts = df['grade'].value_counts()
print(counts)
# Output:
# A    3
# B    2
# C    1

# Get proportions
proportions = df['grade'].value_counts(normalize=True)
print(proportions)
# Output:
# A    0.50
# B    0.33
# C    0.17
```

---

## 🔥 **GroupBy Operations - Complete Explanation**

### **Basic GroupBy Concept**
```python
df.groupby('column')
```
**Purpose:** Groups DataFrame by unique values in specified column(s)
**Returns:** GroupBy object (not a DataFrame yet)
**Think of it as:** "Split-Apply-Combine" operation

### **1. Basic Aggregations**

#### **`.mean()`, `.sum()`, `.count()`**
```python
df.groupby('column').mean()
df.groupby('column').sum()
df.groupby('column').count()
```

**Example with detailed explanation:**
```python
# Sample employee data
employees = pd.DataFrame({
    'department': ['IT', 'HR', 'IT', 'Finance', 'HR', 'IT'],
    'salary': [70000, 60000, 80000, 75000, 65000, 85000],
    'age': [25, 30, 35, 28, 32, 40],
    'experience': [2, 5, 8, 3, 6, 12]
})

# Group by department and calculate mean
dept_avg = employees.groupby('department').mean()
print(dept_avg)
# Output:
#           salary    age  experience
# department                        
# Finance    75000   28.0         3.0
# HR         62500   31.0         5.5
# IT         78333   33.3         7.3

# What happened:
# 1. Split: DataFrame split into groups by department
# 2. Apply: mean() applied to each numeric column in each group
# 3. Combine: Results combined into new DataFrame
```

#### **`.size()` vs `.count()`**
```python
df.groupby('column').size()    # Counts rows in each group (includes NaN)
df.groupby('column').count()   # Counts non-null values in each column
```

**Example:**
```python
df = pd.DataFrame({
    'group': ['A', 'A', 'B', 'B'],
    'value1': [1, 2, None, 4],
    'value2': [10, 20, 30, 40]
})

print(df.groupby('group').size())
# Output:
# group
# A    2
# B    2

print(df.groupby('group').count())
# Output:
#        value1  value2
# group              
# A           2       2
# B           1       2  # Note: value1 has only 1 non-null for group B
```

### **2. Multiple Aggregations with `.agg()`**

#### **Basic `.agg()` usage**
```python
df.groupby('column').agg({
    'salary': ['mean', 'max', 'min'],
    'age': 'count'
})
```

**Detailed Example:**
```python
employees = pd.DataFrame({
    'department': ['IT', 'HR', 'IT', 'Finance', 'HR'],
    'salary': [70000, 60000, 80000, 75000, 65000],
    'age': [25, 30, 35, 28, 32]
})

# Multiple aggregations
result = employees.groupby('department').agg({
    'salary': ['mean', 'max', 'min', 'std'],
    'age': ['count', 'mean']
})

print(result)
# Output: Multi-level column structure
#          salary                           age      
#            mean    max    min        std count  mean
# department                                         
# Finance   75000  75000  75000        NaN     1  28.0
# HR        62500  65000  60000  3535.534     2  31.0
# IT        75000  80000  70000  7071.068     2  30.0
```

#### **Flattening Multi-level Columns**
```python
# Flatten the column names
result.columns = ['_'.join(col).strip() for col in result.columns]
print(result.columns)
# Output: ['salary_mean', 'salary_max', 'salary_min', 'salary_std', 'age_count', 'age_mean']
```

### **3. Custom Functions with `.apply()`**

#### **`.apply()` with lambda functions**
```python
df.groupby('department').apply(lambda x: x.nlargest(2, 'salary'))
```

**Detailed Explanation:**
```python
employees = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'],
    'department': ['IT', 'HR', 'IT', 'IT', 'HR', 'Finance'],
    'salary': [70000, 60000, 80000, 85000, 65000, 75000]
})

# Get top 2 highest paid employees in each department
top_earners = employees.groupby('department').apply(lambda x: x.nlargest(2, 'salary'))
print(top_earners)

# What happens:
# 1. Groups are: IT (Alice, Charlie, David), HR (Bob, Eve), Finance (Frank)
# 2. For each group, nlargest(2, 'salary') is applied
# 3. Results are combined with multi-index
```

#### **Custom aggregation functions**
```python
def salary_range(group):
    return group['salary'].max() - group['salary'].min()

# Apply custom function
salary_ranges = employees.groupby('department').apply(salary_range)
print(salary_ranges)
# Output:
# department
# Finance        0
# HR          5000
# IT         15000
```

---

## 🔗 **Merging and Joining - Complete Guide**

### **1. `pd.merge()` - The Master Function**

#### **Basic Syntax**
```python
pd.merge(left_df, right_df, on='key_column', how='inner')
```

**Parameters Explained:**
- `left`: Left DataFrame
- `right`: Right DataFrame  
- `on`: Column(s) to join on
- `how`: Type of join ('inner', 'left', 'right', 'outer')
- `left_on`/`right_on`: Different column names for joining
- `suffixes`: Suffix for overlapping column names

### **2. Join Types with Examples**

#### **Inner Join (Default)**
```python
pd.merge(df1, df2, on='key', how='inner')
```
**Purpose:** Returns only rows that have matching keys in both DataFrames

**Example:**
```python
# Employee data
employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4],
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'dept_id': [10, 20, 10, 30]
})

# Department data
departments = pd.DataFrame({
    'dept_id': [10, 20, 40],
    'dept_name': ['IT', 'HR', 'Finance']
})

# Inner join
inner_result = pd.merge(employees, departments, on='dept_id', how='inner')
print(inner_result)
# Output:
#    emp_id     name  dept_id dept_name
# 0       1    Alice       10        IT
# 1       2      Bob       20        HR
# 2       3  Charlie       10        IT
# Note: David (dept_id=30) is excluded because dept_id=30 doesn't exist in departments
# Note: Finance (dept_id=40) is excluded because no employee has dept_id=40
```

#### **Left Join**
```python
pd.merge(df1, df2, on='key', how='left')
```
**Purpose:** Returns all rows from left DataFrame, matching rows from right DataFrame

**Example:**
```python
left_result = pd.merge(employees, departments, on='dept_id', how='left')
print(left_result)
# Output:
#    emp_id     name  dept_id dept_name
# 0       1    Alice       10        IT
# 1       2      Bob       20        HR
# 2       3  Charlie       10        IT
# 3       4    David       30       NaN  # David kept, but dept_name is NaN
```

#### **Right Join**
```python
pd.merge(df1, df2, on='key', how='right')
```
**Purpose:** Returns all rows from right DataFrame, matching rows from left DataFrame

**Example:**
```python
right_result = pd.merge(employees, departments, on='dept_id', how='right')
print(right_result)
# Output:
#    emp_id     name  dept_id dept_name
# 0     1.0    Alice       10        IT
# 1     3.0  Charlie       10        IT
# 2     2.0      Bob       20        HR
# 3     NaN      NaN       40   Finance  # Finance dept kept, but no employees
```

#### **Outer Join (Full Join)**
```python
pd.merge(df1, df2, on='key', how='outer')
```
**Purpose:** Returns all rows from both DataFrames

**Example:**
```python
outer_result = pd.merge(employees, departments, on='dept_id', how='outer')
print(outer_result)
# Output:
#    emp_id     name  dept_id dept_name
# 0     1.0    Alice       10        IT
# 1     2.0      Bob       20        HR
# 2     3.0  Charlie       10        IT
# 3     4.0    David       30       NaN  # David with no dept
# 4     NaN      NaN       40   Finance  # Finance with no employees
```

### **3. Advanced Merge Scenarios**

#### **Different Column Names**
```python
pd.merge(df1, df2, left_on='emp_dept', right_on='dept_id')
```

**Example:**
```python
employees = pd.DataFrame({
    'emp_id': [1, 2, 3],
    'name': ['Alice', 'Bob', 'Charlie'],
    'department': [10, 20, 10]  # Different column name
})

departments = pd.DataFrame({
    'dept_id': [10, 20],
    'dept_name': ['IT', 'HR']
})
This documentation provides a solid foundation for working with pandas. Each example builds upon previous concepts, creating a comprehensive learning path from basic Series operations to advanced DataFrame manipulations and data cleaning techniques.
