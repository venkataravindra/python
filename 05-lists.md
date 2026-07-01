# Python Lists - Complete Guide

## What is a List?

A **list** is a collection of elements that can store multiple items in a single variable. Think of it as a container that can hold different types of data.

### Real-World Analogy
- **List** = Shopping cart
- **Elements** = Items in the cart
- **Index** = Position of each item in the cart
- You can add, remove, or change items in the cart

---

## Key Characteristics of Lists

### 1. **Ordered**
Elements have a defined order and maintain that order.

### 2. **Mutable** 
You can change, add, or remove elements after creation.

### 3. **Allow Duplicates**
Same values can appear multiple times.

### 4. **Heterogeneous**
Can store different data types in the same list.

### 5. **Syntax**
Uses square brackets `[]` to define lists.

---

## Creating Lists

### 1. Different Ways to Create Lists

```python
# Empty list
list1 = []
print(f"Empty List: {list1}")  # Output: Empty List: []

# Number list
list2 = [10, 20, 30, 40, 50]
print(f"Number List: {list2}")  # Output: Number List: [10, 20, 30, 40, 50]

# Heterogeneous list (mixed data types)
list3 = [10, "Hello", True, 10.1]
print(f"Heterogeneous List: {list3}")  # Output: Heterogeneous List: [10, 'Hello', True, 10.1]

# Character list from string
list4 = list("Hello")
print(f"Character List: {list4}")  # Output: Character List: ['H', 'e', 'l', 'l', 'o']

# Number list using range
list5 = list(range(5))
print(f"Number List: {list5}")  # Output: Number List: [0, 1, 2, 3, 4]
```

---

## Accessing List Elements

### 1. Indexing and Slicing

```python
list1 = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]

# Basic indexing
print(list1[0])     # Output: 10 (first element)
print(list1[-1])    # Output: 100 (last element)
print(list1[-10])   # Output: 10 (first element using negative index)

# Slicing
print(list1[0:3])   # Output: [10, 20, 30] (elements from index 0 to 2)
print(list1[-10:-7]) # Output: [10, 20, 30] (using negative indices)
print(list1[7:])    # Output: [80, 90, 100] (from index 7 to end)
print(list1[::-1])  # Output: [100, 90, 80, 70, 60, 50, 40, 30, 20, 10] (reverse)
```

### Index Explanation
- **Positive Index**: Starts from 0 (left to right)
- **Negative Index**: Starts from -1 (right to left)
- **Slicing**: `[start:end:step]` - start is included, end is excluded

---

## Modifying Lists (Mutability)

### 1. Changing Elements

```python
list1 = [1, 2, 3, 4, 5]
list1[0] = 1000  # Change first element
print(list1)  # Output: [1000, 2, 3, 4, 5]
```

### 2. Adding Elements

```python
list1 = [1, 2]

# Add single element at the end
list1.append(3)
print(list1)  # Output: [1, 2, 3]

# Add multiple elements
list2 = [4, 5]
list1.extend(list2)
print(list1)  # Output: [1, 2, 3, 4, 5]

# Insert element at specific position
list1.insert(1, 99)  # Insert 99 at index 1
print(list1)  # Output: [1, 99, 2, 3, 4, 5]
```

### 3. Removing Elements

```python
list1 = [1, 2, 3, 2]

# Remove first occurrence of value
list1.remove(2)  # Removes first occurrence of 2
print(list1)  # Output: [1, 3, 2]

# Remove last element
list1.pop()  # Removes last element
print(list1)  # Output: [1, 3]

# Remove element at specific index
list1 = [1, 2, 3, 2]
list1.pop(1)  # Removes element at index 1
print(list1)  # Output: [1, 3, 2]

# Remove all elements
list1.clear()
print(list1)  # Output: []
```

---

## List Methods

### 1. Finding Elements

```python
list1 = [1, 2, 3, 2, 2]

# Find index of first occurrence
print(list1.index(2))  # Output: 1 (first occurrence of 2 is at index 1)

# Count occurrences
print(list1.count(2))  # Output: 3 (2 appears 3 times)
```

### 2. Sorting Lists

```python
list1 = [10, 50, 20, 40, 30]

# Sort in place (modifies original list)
list1.sort()  # Ascending order
print(list1)  # Output: [10, 20, 30, 40, 50]

list1.sort(reverse=True)  # Descending order
print(list1)  # Output: [50, 40, 30, 20, 10]

# Create new sorted list (original unchanged)
list1 = [10, 50, 20, 40, 30]
list2 = sorted(list1)
print(list2)  # Output: [10, 20, 30, 40, 50]
print(list1)  # Output: [10, 50, 20, 40, 30] (unchanged)
```

---

## Iterating Through Lists

### 1. Basic Loop

```python
list1 = [10, 20, 30, 40, 50]

# Loop through elements
for element in list1:
    print(element, end=" ")  # Output: 10 20 30 40 50

print()  # New line

# Loop with index
for i in range(len(list1)):
    print(i, list1[i])
# Output:
# 0 10
# 1 20
# 2 30
# 3 40
# 4 50
```

### 2. Enumerate Function

```python
list1 = [10, 20, 30, 40, 50]

for index, value in enumerate(list1):
    print(f"Index: {index}, Value: {value}")
# Output:
# Index: 0, Value: 10
# Index: 1, Value: 20
# Index: 2, Value: 30
# Index: 3, Value: 40
# Index: 4, Value: 50
```

---

## List Comprehension

**List comprehension** provides a concise way to create lists.

### 1. Basic Comprehension

```python
# Create squares of numbers 0 to 4
squares = [i * i for i in range(5)]
print(squares)  # Output: [0, 1, 4, 9, 16]

# Traditional way (for comparison)
squares_traditional = []
for i in range(5):
    squares_traditional.append(i * i)
print(squares_traditional)  # Output: [0, 1, 4, 9, 16]
```

### 2. Comprehension with Conditions

```python
# Even numbers from 0 to 9
evens = [i for i in range(10) if i % 2 == 0]
print(evens)  # Output: [0, 2, 4, 6, 8]

# Squares of even numbers
even_squares = [i * i for i in range(10) if i % 2 == 0]
print(even_squares)  # Output: [0, 4, 16, 36, 64]
```

### 3. String Processing

```python
# Convert to uppercase
words = ["hello", "world", "python"]
upper_words = [word.upper() for word in words]
print(upper_words)  # Output: ['HELLO', 'WORLD', 'PYTHON']

# Get lengths
word_lengths = [len(word) for word in words]
print(word_lengths)  # Output: [5, 5, 6]
```

---

## Nested Lists

**Nested lists** are lists that contain other lists as elements.

### 1. Creating and Accessing Nested Lists

```python
# 2D list (matrix)
list1 = [[1, 2, 3],
         [4, 5, 6],
         [7, 8, 9]]

# Access elements
print(list1[0][0])  # Output: 1 (first row, first column)
print(list1[1][1])  # Output: 5 (second row, second column)
print(list1[2][2])  # Output: 9 (third row, third column)
```

### 2. Iterating Through Nested Lists

```python
list1 = [[1, 2, 3],
         [4, 5, 6],
         [7, 8, 9]]

# Method 1: Nested loops
for inner_list in list1:
    for element in inner_list:
        print(element, end=" ")
    print()  # New line after each row
# Output:
# 1 2 3 
# 4 5 6 
# 7 8 9 

# Method 2: With indices
for i in range(len(list1)):
    for j in range(len(list1[i])):
        print(f"[{i}][{j}] = {list1[i][j]}")
```

### 3. Practical Example - Student Grades

```python
# Student grades: [Name, Math, Science, English]
students = [
    ["Alice", 85, 92, 78],
    ["Bob", 79, 85, 88],
    ["Charlie", 92, 88, 85]
]

# Calculate average for each student
for student in students:
    name = student[0]
    grades = student[1:]  # All grades except name
    average = sum(grades) / len(grades)
    print(f"{name}: Average = {average:.2f}")
# Output:
# Alice: Average = 85.00
# Bob: Average = 84.00
# Charlie: Average = 88.33
```

---

## List Copying

### 1. Reference Copy (Shallow)

```python
list1 = [1, 2, 3]
list2 = list1  # Reference copy - both point to same list

list1[0] = 100
print(list2)  # Output: [100, 2, 3] (both lists changed)
```

### 2. Deep Copy

```python
list1 = [1, 2, 3]
list2 = list1.copy()  # Deep copy - creates new list
# Alternative: list2 = list(list1)

list1[0] = 100
print(list1)  # Output: [100, 2, 3]
print(list2)  # Output: [1, 2, 3] (unchanged)
```

---

## Common List Operations

### 1. Mathematical Operations

```python
numbers = [10, 20, 30, 40, 50]

print(f"Sum: {sum(numbers)}")        # Output: Sum: 150
print(f"Max: {max(numbers)}")        # Output: Max: 50
print(f"Min: {min(numbers)}")        # Output: Min: 10
print(f"Length: {len(numbers)}")     # Output: Length: 5
```

### 2. Membership Testing

```python
fruits = ["apple", "banana", "orange"]

print("apple" in fruits)      # Output: True
print("grape" in fruits)      # Output: False
print("apple" not in fruits)  # Output: False
```

### 3. Concatenation

```python
list1 = [1, 2, 3]
list2 = [4, 5, 6]

# Using + operator
list3 = list1 + list2
print(list3)  # Output: [1, 2, 3, 4, 5, 6]

# Using extend (modifies original)
list1.extend(list2)
print(list1)  # Output: [1, 2, 3, 4, 5, 6]
```

---

## Practical Examples

### Example 1: Shopping Cart

```python
shopping_cart = []

# Add items
shopping_cart.append("milk")
shopping_cart.append("bread")
shopping_cart.extend(["eggs", "butter"])

print(f"Cart: {shopping_cart}")

# Remove item
shopping_cart.remove("bread")
print(f"After removing bread: {shopping_cart}")

# Check if item exists
if "milk" in shopping_cart:
    print("Milk is in the cart")
```

### Example 2: Grade Management

```python
grades = [85, 92, 78, 96, 88]

# Statistics
print(f"Average: {sum(grades) / len(grades):.2f}")
print(f"Highest: {max(grades)}")
print(f"Lowest: {min(grades)}")

# Add new grade
grades.append(94)
print(f"Updated grades: {grades}")

# Sort grades
grades.sort(reverse=True)
print(f"Sorted (highest first): {grades}")
```

### Example 3: Data Processing

```python
# Process temperature data
temperatures = [23.5, 25.1, 22.8, 26.3, 24.7]

# Convert to Fahrenheit
fahrenheit = [(temp * 9/5) + 32 for temp in temperatures]
print(f"Fahrenheit: {fahrenheit}")

# Find temperatures above average
avg_temp = sum(temperatures) / len(temperatures)
above_avg = [temp for temp in temperatures if temp > avg_temp]
print(f"Above average: {above_avg}")
```

---

## Common Methods Summary

| Method | Description | Example |
|--------|-------------|---------|
| `append(x)` | Add item to end | `list.append(5)` |
| `extend(iterable)` | Add all items from iterable | `list.extend([1,2,3])` |
| `insert(i, x)` | Insert item at position | `list.insert(0, 'first')` |
| `remove(x)` | Remove first occurrence | `list.remove('item')` |
| `pop(i)` | Remove and return item at index | `item = list.pop(0)` |
| `clear()` | Remove all items | `list.clear()` |
| `index(x)` | Find index of first occurrence | `idx = list.index('item')` |
| `count(x)` | Count occurrences | `count = list.count('item')` |
| `sort()` | Sort in place | `list.sort()` |
| `reverse()` | Reverse in place | `list.reverse()` |
| `copy()` | Create shallow copy | `new_list = list.copy()` |

---

## Best Practices

### 1. **Use List Comprehension** for simple transformations
```python
# Good
squares = [x**2 for x in range(10)]

# Less efficient
squares = []
for x in range(10):
    squares.append(x**2)
```

### 2. **Use `enumerate()`** when you need both index and value
```python
# Good
for i, value in enumerate(my_list):
    print(f"{i}: {value}")

# Less readable
for i in range(len(my_list)):
