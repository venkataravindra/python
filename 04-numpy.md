# NumPy (Numerical Python) - Complete Guide

## What is NumPy?

**NumPy** (Numerical Python) is a powerful library for working with arrays and mathematical operations. It's the foundation for data science, machine learning, and scientific computing in Python.

### Why Use NumPy?
- **Fast Performance** - Operations are much faster than regular Python lists
- **Memory Efficient** - Uses less memory than Python lists
- **Mathematical Operations** - Built-in support for complex mathematical operations
- **Broadcasting** - Perform operations on arrays of different sizes
- **Foundation** - Base for other libraries like Pandas, Scikit-learn, etc.

---

## Getting Started

### Installation and Import
```python
# Install: pip install numpy
import numpy as np
```

---

## Creating Arrays

### 1. Basic Array Creation

```python
# 1D Array
arr1 = np.array([10, 20, 30, 40, 50])
print(arr1)  # Output: [10 20 30 40 50]

# 2D Array
arr2 = np.array([[10, 20, 30],
                 [40, 50, 60],
                 [70, 80, 90]])
print(arr2)
```

### 2. Array Properties

```python
arr1 = np.array([10, 20, 30, 40, 50])

print(arr1.ndim)    # Output: 1 (number of dimensions)
print(arr1.shape)   # Output: (5,) (shape of array)
print(arr1.size)    # Output: 5 (total number of elements)
print(arr1.dtype)   # Output: int64 (data type)
```

### 3. Special Array Creation Functions

#### Zeros and Ones
```python
# Array of zeros
arr1 = np.zeros((3, 4))  # 3 rows, 4 columns of zeros
print(arr1)

# Array of ones
arr2 = np.ones((2, 5))   # 2 rows, 5 columns of ones
print(arr2)
```

#### Range Arrays
```python
# Sequential numbers
arr1 = np.arange(10, 21)        # [10, 11, 12, ..., 20]
print(arr1)

arr2 = np.arange(10, 21, 2)     # [10, 12, 14, 16, 18, 20] (step=2)
print(arr2)

# Evenly spaced numbers
arr3 = np.linspace(0, 1, 4)     # [0.0, 0.33, 0.67, 1.0]
print(arr3)
```

---

## Array Manipulation

### 1. Reshaping Arrays

```python
# Convert 1D to 2D
arr1 = np.arange(1, 13)         # [1, 2, 3, ..., 12]
arr2 = arr1.reshape(3, 4)       # 3 rows, 4 columns
print(arr2)
# Output:
# [[ 1  2  3  4]
#  [ 5  6  7  8]
#  [ 9 10 11 12]]
```

### 2. Flattening Arrays

```python
# Convert 2D to 1D
arr1 = np.array([[1, 2], [3, 4]])
arr2 = arr1.flatten()
print(arr2)  # Output: [1 2 3 4]
```

---

## Array Indexing and Slicing

### 1. Basic Indexing

```python
marks = np.array([55, 65, 75, 85, 95])
print(marks[0])     # Output: 55 (first element)
print(marks[-1])    # Output: 95 (last element)
```

### 2. Fancy Indexing

```python
marks = np.array([55, 65, 75, 85, 95])
print(marks[[0, 2, 4]])  # Output: [55 75 95] (elements at indices 0, 2, 4)
```

### 3. Boolean Indexing

```python
# Filter elements based on condition
arr1 = np.array([25000, 50000, 75000, 100000])
arr2 = arr1[arr1 > 50000]  # Elements greater than 50000
print(arr2)  # Output: [75000 100000]
```

---

## Mathematical Operations

### 1. Broadcasting

**Broadcasting** allows operations between arrays of different sizes.

```python
# Add scalar to array
salary = np.array([10000, 20000, 30000, 40000, 50000])
new_salaries = salary + 5000  # Add 5000 to each salary
print(new_salaries)  # Output: [15000 25000 35000 45000 55000]
```

### 2. Array Addition

```python
# 1D Array Addition
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
arr3 = arr1 + arr2
print(arr3)  # Output: [5 7 9]

# 2D Array Addition
arr4 = np.array([[1, 2],
                 [3, 4]])
arr5 = np.array([[5, 6],
                 [7, 8]])
arr6 = arr4 + arr5
print(arr6)
# Output:
# [[ 6  8]
#  [10 12]]
```

### 3. Matrix Multiplication

```python
arr1 = np.array([[1, 2],
                 [3, 4]])
arr2 = np.array([[5, 6],
                 [7, 8]])

arr3 = np.matmul(arr1, arr2)  # Matrix multiplication
print(arr3)
# Output:
# [[19 22]
#  [43 50]]
```

---

## Statistical Operations

### 1. Basic Statistics

```python
marks = np.array([55, 65, 75, 85, 95])

print(marks.sum())   # Output: 375 (sum of all elements)
print(marks.max())   # Output: 95 (maximum value)
print(marks.min())   # Output: 55 (minimum value)
print(marks.mean())  # Output: 75.0 (average)
```

### 2. Axis-wise Operations

```python
arr1 = np.array([[1, 2],
                 [3, 4]])

print(arr1.sum(axis=0))  # Output: [4 6] (column-wise sum)
print(arr1.sum(axis=1))  # Output: [3 7] (row-wise sum)
```

### 3. Finding Indices

```python
arr1 = np.array([10, 50, 20, 40, 30])
print(arr1.argmax())  # Output: 1 (index of maximum value)
print(arr1.argmin())  # Output: 0 (index of minimum value)
```

---

## Conditional Operations

### 1. Where Function

```python
# Replace values based on condition
marks = np.array([55, 65, 75, 85, 95])
result = np.where(marks > 60, "pass", "fail")
print(result)  # Output: ['fail' 'pass' 'pass' 'pass' 'pass']

# Replace negative values with 0
profits = np.array([100, -50, 30, -10, 1000, -90])
new_arr = np.where(profits < 0, 0, profits)
print(new_arr)  # Output: [100   0  30   0 1000   0]
```

---

## Array Sorting and Searching

### 1. Sorting

```python
arr1 = np.array([10, 50, 20, 40, 30])
arr2 = np.sort(arr1)  # Returns sorted copy
print(arr2)  # Output: [10 20 30 40 50]

# Find top 3 elements
top = np.sort(arr1)[-3:]  # Last 3 elements after sorting
print(top)  # Output: [30 40 50]
```

### 2. Reversing Arrays

```python
arr1 = np.array([10, 20, 30, 40, 50])
new_arr = arr1[::-1]  # Reverse using slicing
print(new_arr)  # Output: [50 40 30 20 10]
```

---

## Advanced Operations

### 1. Finding Duplicates

```python
arr1 = np.array([10, 20, 30, 10, 20, 30, 40])
uniques, counts = np.unique(arr1, return_counts=True)
print(uniques)  # Output: [10 20 30 40]
print(counts)   # Output: [2 2 2 1]
```

### 2. Counting Elements

```python
# Count even numbers
arr1 = np.array([10, 15, 20, 25, 30, 35])
count_even = np.count_nonzero(arr1 % 2 == 0)
print(count_even)  # Output: 3
```

### 3. Array Copying

```python
arr1 = np.array([1, 2, 3])

# Shallow copy (changes affect both)
arr2 = arr1
arr1[0] = 100
print(arr2)  # Output: [100   2   3]

# Deep copy (independent arrays)
arr1 = np.array([1, 2, 3])
arr2 = arr1.copy()
arr1[0] = 100
print(arr2)  # Output: [1 2 3]
```

---

## Array Concatenation and Splitting

### 1. Concatenation

```python
# Concatenate 1D arrays
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
new_arr = np.concatenate((arr1, arr2))
print(new_arr)  # Output: [1 2 3 4 5 6]
```

### 2. Vertical and Horizontal Stacking

```python
# Vertical stacking
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
arr3 = np.vstack((arr1, arr2))
print(arr3)
# Output:
# [[1 2 3]
#  [4 5 6]]

# Horizontal stacking
arr1 = np.array([[1], [2], [3]])
arr2 = np.array([[4], [5], [6]])
arr3 = np.hstack((arr1, arr2))
print(arr3)
# Output:
# [[1 4]
#  [2 5]
#  [3 6]]
```

### 3. Splitting Arrays

```python
arr1 = np.arange(12)  # [0, 1, 2, ..., 11]
parts = np.split(arr1, 3)  # Split into 3 equal parts
print(parts)  # [array([0, 1, 2, 3]), array([4, 5, 6, 7]), array([8, 9, 10, 11])]
```

---

## Special Arrays and Operations

### 1. Identity Matrix

```python
arr1 = np.identity(4)  # 4x4 identity matrix
print(arr1)
# Output:
# [[1. 0. 0. 0.]
#  [0. 1. 0. 0.]
#  [0. 0. 1. 0.]
#  [0. 0. 0. 1.]]
```

### 2. Diagonal Operations

```python
arr1 = np.array([[1, 2, 3],
                 [4, 5, 6],
                 [7, 8, 9]])
arr2 = np.diag(arr1)  # Extract diagonal elements
print(arr2)  # Output: [1 5 9]
```

### 3. Dot Product

```python
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
result = np.dot(arr1, arr2)  # (1*4) + (2*5) + (3*6)
print(result)  # Output: 32
```

---

## Data Normalization

```python
# Normalize data to 0-1 range
marks = np.array([50, 60, 70, 80, 90])
normalization = (marks - marks.min()) / (marks.max() - marks.min())
print(normalization)  # Output: [0.   0.25 0.5  0.75 1.  ]
```

---

## Random Number Generation

```python
# Set seed for reproducible results
np.random.seed(10)
arr1 = np.random.randint(1, 100, 10)  # 10 random integers between 1-99
print(arr1)
```

---

## Memory and Performance Considerations

### 1. View vs Copy

```python
# Reshape creates a view (shares memory)
arr1 = np.arange(1, 10)
arr2 = arr1.reshape(3, 3)  # View of arr1

arr1[0] = 100    # Changes affect both arrays
arr2[2][2] = 900 # Changes affect both arrays
print(arr1)      # Both arrays are modified
```

---

## Practical Examples

### Example 1: Salary Analysis
```python
# Find high salaries
salaries = np.array([10000, 30000, 20000, 50000, 90000, 15000])
high_salaries = salaries[salaries > 50000]
print(high_salaries)  # Output: [90000]
```

### Example 2: Grade Processing
```python
# Process student grades
grades = np.array([85, 92, 78, 96, 88])
print(f"Average: {grades.mean():.2f}")
print(f"Highest: {grades.max()}")
print(f"Students above average: {np.sum(grades > grades.mean())}")
```

### Example 3: Matrix Operations
```python
# Calculate correlation matrix
data = np.array([[1, 2, 3],
                 [4, 5, 6],
                 [7, 8, 9]])
correlation = np.corrcoef(data)
print(correlation)
```

---

## Key Benefits Summary

### Performance Advantages
- **Speed**: 10-100x faster than Python lists
- **Memory**: Uses less memory than lists
- **Vectorization**: Operations on entire arrays at once

### Functionality
- **Mathematical Operations**: Built-in math functions
- **Broadcasting**: Operations between different sized arrays
- **Indexing**: Powerful selection and filtering
- **Integration**: Works seamlessly with other libraries

### Best Practices
1. **Use NumPy arrays** instead of Python lists for numerical data
2. **Vectorize operations** instead of using loops
3. **Use appropriate data types** to save memory
4. **Understand view vs copy** to avoid unexpected behavior
5. **Use broadcasting** for efficient operations

This comprehensive guide covers all essential NumPy operations with practical examples that demonstrate real-world usage patterns for data science and scientific computing.
