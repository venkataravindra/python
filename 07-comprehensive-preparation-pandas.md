# Comprehensive Pandas Interview Preparation Guide

Based on your current notes, you have a good foundation, but there are **many critical pandas functions missing** that are frequently asked in interviews. Let me provide a complete roadmap:

## 🔍 **What You're Missing (Interview Critical Functions)**

### **1. Advanced Data Manipulation**
```python
# Sorting (Very Common in Interviews)
df.sort_values('column_name')
df.sort_values(['col1', 'col2'], ascending=[True, False])
df.sort_index()

# Ranking
df.rank()
df['column'].rank(method='dense')

# Unique values and counts
df['column'].unique()
df['column'].nunique()
df['column'].value_counts()
df['column'].value_counts(normalize=True)  # Percentages
```

### **2. GroupBy Operations (Interview Favorite)**
```python
# Basic grouping
df.groupby('column').mean()
df.groupby('column').sum()
df.groupby('column').count()
df.groupby('column').size()

# Multiple grouping
df.groupby(['col1', 'col2']).agg({
    'salary': ['mean', 'max', 'min'],
    'age': 'count'
})

# Custom aggregations
df.groupby('department').apply(lambda x: x.nlargest(2, 'salary'))
```

### **3. Merging and Joining (Must Know)**
```python
# Different types of joins
pd.merge(df1, df2, on='key')
pd.merge(df1, df2, on='key', how='left')   # left join
pd.merge(df1, df2, on='key', how='right')  # right join
pd.merge(df1, df2, on='key', how='outer')  # full outer join
pd.merge(df1, df2, on='key', how='inner')  # inner join

# Concatenation
pd.concat([df1, df2])
pd.concat([df1, df2], axis=1)  # side by side
```

### **4. String Operations (Common Interview Questions)**
```python
# String methods
df['name'].str.upper()
df['name'].str.lower()
df['name'].str.contains('pattern')
df['name'].str.startswith('A')
df['name'].str.endswith('son')
df['name'].str.len()
df['name'].str.split(' ')
df['name'].str.replace('old', 'new')
df['name'].str.extract('(\d+)')  # Extract numbers
```

### **5. Date/Time Operations (Frequently Asked)**
```python
# Converting to datetime
df['date'] = pd.to_datetime(df['date'])

# Date operations
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['weekday'] = df['date'].dt.day_name()

# Date filtering
df[df['date'] > '2023-01-01']
df[df['date'].between('2023-01-01', '2023-12-31')]
```

### **6. Pivot Tables and Reshaping (Advanced)**
```python
# Pivot tables
df.pivot_table(values='salary', index='department', columns='gender', aggfunc='mean')

# Melting (wide to long)
pd.melt(df, id_vars=['name'], value_vars=['math', 'science'])

# Crosstab
pd.crosstab(df['department'], df['gender'])
```

---

## 🎯 **Top 50 Interview Questions with Solutions**

### **Basic Level Questions**

#### **Q1: How to find duplicate rows?**
```python
# Find duplicates
df.duplicated()
df[df.duplicated()]

# Remove duplicates
df.drop_duplicates()
df.drop_duplicates(subset=['column'])
```

#### **Q2: How to rename columns?**
```python
# Rename specific columns
df.rename(columns={'old_name': 'new_name'})

# Rename all columns
df.columns = ['new_col1', 'new_col2', 'new_col3']
```

#### **Q3: How to reset index?**
```python
df.reset_index()
df.reset_index(drop=True)  # Don't keep old index as column
```

### **Intermediate Level Questions**

#### **Q4: Find top N values in each group**
```python
# Top 2 salaries in each department
df.groupby('department').apply(lambda x: x.nlargest(2, 'salary'))
```

#### **Q5: Calculate percentage of total**
```python
df['salary_pct'] = df['salary'] / df['salary'].sum() * 100
```

#### **Q6: Find rows with maximum value in a group**
```python
df.loc[df.groupby('department')['salary'].idxmax()]
```

### **Advanced Level Questions**

#### **Q7: Create age groups**
```python
df['age_group'] = pd.cut(df['age'], bins=[0, 25, 35, 50, 100], 
                        labels=['Young', 'Adult', 'Middle', 'Senior'])
```

#### **Q8: Rolling calculations**
```python
df['rolling_avg'] = df['salary'].rolling(window=3).mean()
```

#### **Q9: Conditional column creation**
```python
df['performance'] = np.where(df['score'] > 80, 'High', 
                   np.where(df['score'] > 60, 'Medium', 'Low'))
```

---

## 🔥 **Most Common Interview Scenarios**

### **Scenario 1: Employee Data Analysis**
```python
# Sample data
employees = pd.DataFrame({
    'name': ['John', 'Jane', 'Bob', 'Alice', 'Charlie'],
    'department': ['IT', 'HR', 'IT', 'Finance', 'IT'],
    'salary': [70000, 65000, 80000, 75000, 85000],
    'age': [25, 30, 35, 28, 32],
    'join_date': ['2020-01-15', '2019-03-20', '2018-07-10', '2021-02-28', '2017-11-05']
})

# Convert date
employees['join_date'] = pd.to_datetime(employees['join_date'])

# Common interview questions:
# 1. Average salary by department
employees.groupby('department')['salary'].mean()

# 2. Highest paid employee in each department
employees.loc[employees.groupby('department')['salary'].idxmax()]

# 3. Employees joined in last 2 years
cutoff_date = pd.Timestamp.now() - pd.DateOffset(years=2)
employees[employees['join_date'] > cutoff_date]

# 4. Department with highest average salary
employees.groupby('department')['salary'].mean().idxmax()
```

### **Scenario 2: Sales Data Analysis**
```python
# Sample sales data
sales = pd.DataFrame({
    'date': pd.date_range('2023-01-01', periods=100),
    'product': np.random.choice(['A', 'B', 'C'], 100),
    'sales': np.random.randint(100, 1000, 100),
    'region': np.random.choice(['North', 'South', 'East', 'West'], 100)
})

# Interview questions:
# 1. Monthly sales trend
sales.groupby(sales['date'].dt.month)['sales'].sum()

# 2. Best performing product by region
sales.groupby(['region', 'product'])['sales'].sum().unstack()

# 3. Moving average
sales['moving_avg'] = sales['sales'].rolling(window=7).mean()
```

---

## 📚 **Functions You Must Master for Interviews**

### **Data Exploration (Must Know)**
```python
df.info()           # Data types and memory usage
df.describe()       # Statistical summary
df.shape           # Dimensions
df.columns         # Column names
df.index           # Index information
df.dtypes          # Data types
df.memory_usage()  # Memory consumption
```

### **Data Cleaning (Critical)**
```python
df.isnull().sum()           # Count missing values
df.dropna()                 # Remove missing values
df.fillna()                 # Fill missing values
df.drop_duplicates()        # Remove duplicates
df.replace()                # Replace values
df.astype()                 # Change data types
```

### **Data Selection (Essential)**
```python
df.loc[]           # Label-based selection
df.iloc[]          # Position-based selection
df.query()         # SQL-like filtering
df.where()         # Conditional selection
df.isin()          # Check membership
```

### **Data Aggregation (Very Important)**
```python
df.groupby()       # Group operations
df.agg()           # Multiple aggregations
df.transform()     # Transform within groups
df.apply()         # Apply functions
df.pivot_table()   # Pivot operations
```

---

## 🎯 **Interview Tips and Tricks**

### **1. Performance Questions**
```python
# Use vectorized operations instead of loops
# Good
df['new_col'] = df['col1'] * df['col2']

# Bad
for i in range(len(df)):
    df.loc[i, 'new_col'] = df.loc[i, 'col1'] * df.loc[i, 'col2']
```

### **2. Memory Optimization**
```python
# Check memory usage
df.info(memory_usage='deep')

# Optimize data types
df['category_col'] = df['category_col'].astype('category')
df['int_col'] = pd.to_numeric(df['int_col'], downcast='integer')
```

### **3. Chaining Operations**
```python
# Method chaining (impressive in interviews)
result = (df
          .groupby('department')
          .agg({'salary': 'mean', 'age': 'count'})
          .reset_index()
          .sort_values('salary', ascending=False)
          .head(5))
```

---

## 🚀 **Practice Problems for Interview Prep**

### **Problem 1: Data Cleaning Challenge**
```python
# Create messy data
messy_data = pd.DataFrame({
    'Name': ['John Doe', 'jane smith', 'BOB JOHNSON', None, 'Alice Brown'],
    'Email': ['john@email.com', 'JANE@EMAIL.COM', 'bob@email', 'alice@email.com', None],
    'Salary': ['50000', '60000', 'invalid', '70000', '80000'],
    'Department': ['IT', 'it', 'HR', 'hr', 'Finance']
})

# Clean this data:
# 1. Standardize name format
# 2. Validate email addresses
# 3. Convert salary to numeric
# 4. Standardize department names
```

### **Problem 2: Sales Analysis**
```python
# Analyze this sales data and answer:
# 1. Which month had highest sales?
# 2. Which product category is most profitable?
# 3. Calculate month-over-month growth
# 4. Find seasonal trends
```

---

## 📖 **Study Plan for Interview Success**

### **Week 1: Master the Basics**
- Data structures (Series, DataFrame)
- Data loading and exploration
- Basic filtering and selection

### **Week 2: Data Manipulation**
- GroupBy operations
- Merging and joining
- String operations

### **Week 3: Advanced Operations**
- Pivot tables and reshaping
- Date/time operations
- Window functions

### **Week 4: Practice and Performance**
- Solve real-world problems
- Optimize code performance
- Practice explaining your solutions

---

## 🎯 **Final Interview Checklist**

✅ **Can you explain the difference between loc and iloc?**
✅ **Can you perform complex groupby operations?**
✅ **Can you merge datasets using different join types?**
✅ **Can you handle missing data effectively?**
✅ **Can you work with datetime data?**
✅ **Can you create pivot tables?**
✅ **Can you optimize pandas code for performance?**
✅ **Can you explain when to use apply vs transform?**

Master these concepts, and you'll be well-prepared for any pandas interview question!
