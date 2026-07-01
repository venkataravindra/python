# Python Variables and Data Types - Complete Guide

## What are Variables?

**Variables** are containers used to **store data** in memory. Think of them as labeled boxes where you can put different types of information.

```python
name = "John"        # String variable
age = 25            # Integer variable
is_student = True   # Boolean variable
```

---

## Python Data Types Overview

Python supports 8 main data types:

1. **String** - Text data
2. **Integer** - Whole numbers
3. **Float** - Decimal numbers
4. **Boolean** - True/False values
5. **List** - Ordered collection (mutable)
6. **Tuple** - Ordered collection (immutable)
7. **Dictionary** - Key-value pairs
8. **Set** - Unique elements collection
9. **None** - Empty/no value

---

## 1. STRING Data Type

### Definition
A **string** is a collection of characters. It represents text data.

### Three Ways to Define Strings:

```python
# 1. Double quotes
name = "Python"

# 2. Single quotes  
language = 'Python'

# 3. Triple quotes (for paragraphs)
course = """
    Here, we will cover:
    1) Python
    2) Machine Learning
    3) Deep Learning
"""
```

### String Indexing and Slicing

```python
str = "Python"
# P=0/-6, y=1/-5, t=2/-4, h=3/-3, o=4/-2, n=5/-1

print(str[2])     # Output: t
print(str[-4])    # Output: t
print(str[0:2])   # Output: Py (0 included, 2 excluded)
print(str[:3])    # Output: Pyt (start to index 3)
print(str[2:])    # Output: thon (index 2 to end)
print(str[::-1])  # Output: nohtyP (reverse string)
print(str[::-2])  # Output: nhy (reverse with step 2)
```

### String Properties and Methods

#### Immutable Nature
```python
str = "Hello"
# str[0] = "h"  # This will cause TypeError!
# Correct way to modify:
new_str = "h" + str[1:]  # Output: hello
```

#### String Operations
```python
str = "VPro"
str1 = str * 3          # Output: VProVProVPro

# Counting characters
str = "aaaa"
print(str.count("a"))   # Output: 4
print(str.count("aa"))  # Output: 3

# Replacing characters
str = "banana"
print(str.replace("a", "A"))      # Output: bAnAnA
print(str.replace("a", "A", 1))   # Output: bAnana (replace only first)
```

---

## 2. NUMERIC Data Types

### Types of Numbers
1. **int** - Whole numbers
2. **float** - Decimal numbers  
3. **complex** - Complex numbers

### Integer Operations
```python
num1 = 2000
num2 = 1000
add = num1 + num2       # Addition: 3000
sub = num1 - num2       # Subtraction: 1000
print(f"Addition: {add}")
print(f"Subtraction: {sub}")
```

### Mathematical Operations
```python
print(10 / 2)    # Division: 5.0
print(10 // 3)   # Floor division: 3
print(10 % 3)    # Modulus: 1
print(2 ** 5)    # Power: 32
```

### Type Conversion
```python
# Int to Float
num = 10
print(float(num))       # Output: 10.0

# Float to Int
num = 10.6
print(int(num))         # Output: 10

# String to Number
str = "100"
print(int(str))         # Output: 100
print(float(str))       # Output: 100.0
```

### Complex Numbers
```python
c = 5 + 4j
print(c.real)           # Output: 5.0
print(c.imag)           # Output: 4.0
print(type(c))          # Output: <class 'complex'>
```

### Floating Point Precision Issue
```python
print(0.1 + 0.2)        # Output: 0.30000000000000004
print(0.1 + 0.2 == 0.3) # Output: False

# Solution:
import math
print(math.isclose(0.1 + 0.2, 0.3))  # Output: True
```

### Number Systems
```python
num1 = 100              # Decimal
num2 = 0x123ABC         # Hexadecimal
num3 = 0o123            # Octal  
num4 = 0b1010           # Binary
print(num2)             # Output: 1194684
print(num3)             # Output: 83
print(num4)             # Output: 10
```

---

## 3. BOOLEAN Data Type

### Definition
Boolean represents **True** or **False** values.

```python
flag = True
flag1 = False
print(flag)             # Output: True
print(flag1)            # Output: False
```

### Boolean Arithmetic
```python
print(True + True)      # Output: 2 (True = 1)
print(1 + True)         # Output: 2
print(1 + True + 0 + False)  # Output: 2
# print(True / False)   # This causes ZeroDivisionError
print(False / True)     # Output: 0.0
```

### Conditional Expressions
```python
flag = True
result = "Gen AI" if flag else "Agentic AI"
print(result)           # Output: Gen AI

flag1 = False
result1 = "Gen AI" if flag1 else "Agentic AI"
print(result1)          # Output: Agentic AI
```

---

## 4. LIST Data Type

### Properties
- **Ordered** - Elements have a defined order
- **Mutable** - Can be modified after creation
- **Heterogeneous** - Can contain different data types
- **Syntax:** `[]`
- **Indexing:** Starts from 0

### List Operations
```python
list1 = [10, 20, 30, 40, 50]
print(list1[0])         # Output: 10
print(list1[-5])        # Output: 10
print(list1[0:3])       # Output: [10, 20, 30]
print(list1[-3:-1])     # Output: [30, 40]
print(list1[3:])        # Output: [40, 50]
print(list1[:2])        # Output: [10, 20]
print(list1[::-1])      # Output: [50, 40, 30, 20, 10]
print(list1[::-2])      # Output: [50, 30, 10]
```

### List Mutability
```python
import sys
list2 = [10, "Hello", True, 10.1, None]
list2[0] = 1000         # Modifying element
print(list2)            # Output: [1000, 'Hello', True, 10.1, None]
print(sys.getsizeof(list2))  # Memory size
```

---

## 5. TUPLE Data Type

### Properties
- **Ordered** - Elements have a defined order
- **Immutable** - Cannot be modified after creation
- **Heterogeneous** - Can contain different data types
- **Syntax:** `()`
- **Memory Efficient** - Uses less memory than lists
- **Faster Operations** - Due to hashtable implementation

### Memory Comparison
```python
import sys
list1 = [10, 20, 30, 40, 50]
tuple1 = (10, 20, 30, 40, 50)
print(sys.getsizeof(list1))   # Larger memory
print(sys.getsizeof(tuple1))  # Smaller memory
```

### Tuple Immutability
```python
tuple1 = (10, 20, 30, 40, 50)
# tuple1[2] = 3000  # This causes TypeError!
```

### Converting Between List and Tuple
```python
tuple1 = (10, 20, 30, 40, 50)
# Tuple to List
list1 = list(tuple1)
list1[0] = 1000
# List to Tuple
tuple2 = tuple(list1)
print(tuple2)           # Output: (1000, 20, 30, 40, 50)
```

---

## 6. DICTIONARY Data Type

### Properties
- **Key-Value Pairs** - Data stored as key: value
- **Keys are Immutable** - Cannot modify keys
- **Values are Mutable** - Can modify values
- **Syntax:** `{}`

### Dictionary Operations
```python
d1 = {
    "name": "vpro",
    "subject": "Agentic AI"
}
print(d1)               # Output: {'name': 'vpro', 'subject': 'Agentic AI'}
print(d1.keys())        # Output: dict_keys(['name', 'subject'])
print(d1.values())      # Output: dict_values(['vpro', 'Agentic AI'])
print(d1.items())       # Output: dict_items([('name', 'vpro'), ('subject', 'Agentic AI')])
```

### Modifying Dictionaries
```python
d1 = {"name": "Agentic"}
d1["name"] = "Agentic AI"           # Modify existing key
d1["new_subject"] = "Quantum Computing"  # Add new key
print(d1)               # Output: {'name': 'Agentic AI', 'new_subject': 'Quantum Computing'}
```

### Dictionary Methods
```python
d1 = {}
d1["key1"] = 100
d1["key2"] = 200
d1["key3"] = 300
d1["key1"] = 1000       # Update value
d1.pop("key2")          # Remove specific key
d1.popitem()            # Remove last item
print(d1)               # Output: {'key1': 1000}
```

---

## 7. SET Data Type

### Properties
- **No Duplicates** - Automatically removes duplicate values
- **Unordered** - No defined order
- **Syntax:** `{}`

### Set Creation
```python
s1 = {10, 20, 30, 10, 20, 30}
print(s1)               # Output: {10, 20, 30}

s2 = set([10, 20, 30, 10, 20, 30])
print(s2)               # Output: {10, 20, 30}

s3 = set((10, 20, 10, 20))
print(s3)               # Output: {10, 20}
```

---

## 8. NONE Data Type

### Definition
**None** represents the absence of a value or empty value.

```python
x = None
print(x)                # Output: None
print(type(x))          # Output: <class 'NoneType'>
```

---

## String Formatting Methods

### 1. Old Style Formatting (%)
```python
# %s - string, %d - integer, %f - float
name = "VPro"
print("I am the founder of %s" % name)

subject = "Agentic AI"
version = 2
print("Current Trending Subject is %s and Version is %d" % (subject, version))

fever = 97.5
print("Fever = %f" % fever)         # Output: Fever = 97.500000
print("Fever = %.2f" % fever)       # Output: Fever = 97.50
```

### 2. .format() Method
```python
name = "VPro"
year = 2025
print("Name is {} and established in {}".format(name, year))
print("Name is {1} and established in {0}".format(year, name))  # Position-based
```

### 3. f-strings (Modern Python)
```python
name = "VPro"
year = 2025
print(f"Name is {name} and established in {year}")
```

---

## Advanced Features

### Rich Library for Colored Output
```python
from rich import print
print("[red]Hello[/red]")
print("[bold green]VPro[/bold green]")
```

### List Multiplication
```python
print(["a"] * 3)        # Output: ['a', 'a', 'a']
```

---

## Key Takeaways

### Memory and Performance
- **Tuples** use less memory than lists
- **Tuples** have faster operations than lists
- **Sets** automatically handle duplicates

### Mutability Summary
- **Mutable:** Lists, Dictionaries, Sets
- **Immutable:** Strings, Tuples, Integers, Floats, Booleans

### When to Use What
- **Lists:** When you need ordered, changeable data
- **Tuples:** When you need ordered, unchangeable data
- **Dictionaries:** When you need key-value relationships
- **Sets:** When you need unique elements only
- **Strings:** For text data

This comprehensive guide covers all fundamental Python data types with practical examples that any beginner can understand and apply in real-world programming scenarios.
