# Python Functions - Complete Guide

## What are Functions?

A **function** is a block of code that performs a specific task or contains particular **business logic**. Think of it as a reusable piece of code that you can call whenever needed.

### Why Use Functions?
- **Reusability** - Write once, use multiple times
- **Organization** - Break large programs into smaller, manageable pieces
- **Maintainability** - Easy to update and debug
- **Readability** - Makes code more understandable

---

## Function Declaration

### Basic Syntax
```python
def function_name():
    # Function body
    pass  # 'pass' keyword represents empty function
```

**Key Points:**
- `def` is the keyword used to **declare** a function
- `pass` is used to represent an **empty function**

---

## Types of Functions

There are **4 main types** of functions based on parameters and return values:

### 1. No Parameters - No Return Type

```python
def addition():
    num1 = 200
    num2 = 100
    res = num1 + num2
    print(f"Addition: {res}")

# Function call
addition()  # Output: Addition: 300
```

**Use Case:** When you have fixed values and just want to perform an action.

### 2. No Parameters - With Return Type

```python
def addition():
    num1 = 200
    num2 = 100
    res = num1 + num2
    return res

# Function call
x = addition()
print(x)  # Output: 300
```

**Use Case:** When you have fixed values but want to use the result elsewhere.

### 3. With Parameters - No Return Type

```python
def addition(num1, num2):
    res = num1 + num2
    print(f"Addition: {res}")

# Function call
addition(200, 100)  # Output: Addition: 300
```

**Use Case:** When you want to pass different values but just perform an action.

### 4. With Parameters - With Return Type

```python
def addition(num1, num2):
    res = num1 + num2
    return res

# Function call
x = addition(200, 100)
print(f"Addition: {x}")  # Output: Addition: 300
```

**Use Case:** Most flexible - can pass different values and use the result.

---

## Parameter Types

### 1. Default Parameters

Default parameters have predefined values that are used when no argument is passed.

```python
def test_func(num1=200, num2=100):
    res = num1 + num2
    print(f"Addition: {res}")

# Different ways to call:
test_func()                    # Uses default values: 200 + 100 = 300
test_func(2000, 1000)         # Uses provided values: 2000 + 1000 = 3000
test_func(2000)               # num1=2000, num2=100 (default)
test_func(num2=1000)          # num1=200 (default), num2=1000
test_func(num1=1)             # num1=1, num2=100 (default)
```

### 2. Regular + Default Parameters

```python
def test_func(param1, param2, param3="Hello"):
    print(param1, param2, param3)

test_func(100, 200)                    # Output: 100 200 Hello
test_func(100, 200, 300)               # Output: 100 200 300
test_func(param3="Agentic AI", param2="Gen AI", param1="Python")  # Named arguments
```

**Important Rule:** Regular parameters must come before default parameters.

### 3. Variable-Length Arguments (*args)

Use `*` to accept any number of arguments. The parameter becomes a **tuple**.

```python
def test_func(*param1):
    print(param1)
    print(type(param1))  # <class 'tuple'>

test_func("Gen AI")                    # Output: ('Gen AI',)
test_func("Gen AI", "Agentic AI")      # Output: ('Gen AI', 'Agentic AI')
test_func(10, 20, 30, 40, 50)         # Output: (10, 20, 30, 40, 50)
```

**Limitation:** Only **one** variable-length argument is allowed per function.

#### Practical Example:
```python
def test_func(*subjects):
    for sub in subjects:
        print(sub, end=" ")

test_func("Gen AI", "Agentic AI", "RAG", "MCP", "MCP Client")
# Output: Gen AI Agentic AI RAG MCP MCP Client
```

### 4. Keyword Arguments (**kwargs)

Use `**` to accept any number of keyword arguments. The parameter becomes a **dictionary**.

```python
def test_func(**param1):
    print(param1)
    print(type(param1))  # <class 'dict'>

test_func(name="VPro", sub="Gen AI")
# Output: {'name': 'VPro', 'sub': 'Gen AI'}
```

### 5. Combining All Parameter Types

```python
def test_func(param1, param2="Hello", *param3, **param4):
    print(f"Regular: {param1}")
    print(f"Default: {param2}")
    print(f"Variable-length: {param3}")
    print(f"Keyword: {param4}")

test_func(100)
# Output: Regular: 100, Default: Hello, Variable-length: (), Keyword: {}

test_func(100, 200, 300, 400, 500, num1=600, num2=700)
# Output: Regular: 100, Default: 200, Variable-length: (300, 400, 500), Keyword: {'num1': 600, 'num2': 700}
```

**Parameter Order Rule:**
1. Regular parameters
2. Default parameters  
3. Variable-length arguments (*args)
4. Keyword arguments (**kwargs)

---

## Lambda Functions (Anonymous Functions)

**Lambda functions** are small, anonymous functions that can have any number of arguments but can only have one expression.

### Basic Syntax
```python
lambda arguments: expression
```

### Examples

#### Single Parameter
```python
x = lambda num1: num1 * num1
print(x(10))  # Output: 100
```

#### Multiple Parameters
```python
x = lambda num1, num2: num1 + num2
print(x(200, 100))  # Output: 300
```

#### Conditional Lambda
```python
x = lambda num1: "Even" if num1 % 2 == 0 else "Odd"
print(x(2))   # Output: Even
print(x(3))   # Output: Odd
```

#### Three Parameters
```python
x = lambda num1, num2, num3: num1 * num2 * num3
print(x(10, 20, 30))  # Output: 6000
```

### Lambda with Built-in Functions

#### 1. sorted() with Lambda
```python
subs = ["GenAI", "AgenticAI", "Quantum", "RAG"]
# Sort by length of strings
res = sorted(subs, key=lambda x: len(x), reverse=True)
print(res)  # Output: ['AgenticAI', 'Quantum', 'GenAI', 'RAG']
```

#### 2. Employee Sorting Example
```python
emps = [("Emp1", 50000), ("Emp2", 30000), ("Emp3", 51000)]

# Sort by salary (mutable - modifies original)
emps.sort(key=lambda x: x[1])
print(emps)  # Output: [('Emp2', 30000), ('Emp1', 50000), ('Emp3', 51000)]

# Sort by salary (immutable - creates new list)
res = sorted(emps, key=lambda x: x[1])
print(res)
```

---

## Higher-Order Functions

### 1. map() Function

Applies a function to every item in an iterable.

```python
# Square each number
result = list(map(lambda x: x * x, (1, 2, 3, 4, 5)))
print(result)  # Output: [1, 4, 9, 16, 25]

# Convert to half values
result = list(map(lambda x: x * 0.5, [1, 2, 3, 4, 5]))
print(result)  # Output: [0.5, 1.0, 1.5, 2.0, 2.5]
```

### 2. filter() Function

Filters items from an iterable based on a condition.

```python
# Filter even numbers
result = list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]))
print(result)  # Output: [2, 4, 6, 8, 10]

# Filter odd numbers
result = list(filter(lambda x: x % 2 != 0, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]))
print(result)  # Output: [1, 3, 5, 7, 9]
```

### 3. reduce() Function

Applies a function cumulatively to items in an iterable to reduce it to a single value.

```python
from functools import reduce

# Sum of all elements
result = reduce(lambda num1, num2: num1 + num2, [1, 2, 3, 4, 5])
print(result)  # Output: 15

# Product of all elements
result = reduce(lambda num1, num2: num1 * num2, [1, 2, 3, 4, 5])
print(result)  # Output: 120
```

---

## Complex Example: Combining map(), filter(), and reduce()

```python
from functools import reduce

# Step by step:
numbers = [10, 20, 30, 40, 50]

# 1. Square each number using map()
squared = list(map(lambda x: x * x, numbers))
print(f"Squared: {squared}")  # [100, 400, 900, 1600, 2500]

# 2. Filter numbers greater than 900 using filter()
filtered = list(filter(lambda x: x > 900, squared))
print(f"Filtered: {filtered}")  # [1600, 2500]

# 3. Sum the filtered numbers using reduce()
final_sum = reduce(lambda x, y: x + y, filtered)
print(f"Final Sum: {final_sum}")  # 4100

# All in one line:
result = reduce(
    lambda num1, num2: num1 + num2,
    list(filter(
        lambda num1: num1 > 900,
        list(map(lambda num1: num1 * num1, [10, 20, 30, 40, 50]))
    ))
)
print(f"One-liner result: {result}")  # 4100
```

---

## Practical Exercises

### Exercise 1: Square of a Number
```python
def square(num):
    return num * num

print(square(5))  # Output: 25

# Lambda version
square_lambda = lambda x: x * x
print(square_lambda(5))  # Output: 25
```

### Exercise 2: Largest Among Two Numbers
```python
def largest_two(a, b):
    return a if a > b else b

print(largest_two(10, 20))  # Output: 20

# Lambda version
largest_lambda = lambda a, b: a if a > b else b
print(largest_lambda(10, 20))  # Output: 20
```

### Exercise 3: Largest Among Three Numbers
```python
def largest_three(a, b, c):
    return max(a, b, c)

print(largest_three(10, 30, 20))  # Output: 30

# Lambda version
largest_three_lambda = lambda a, b, c: max(a, b, c)
print(largest_three_lambda(10, 30, 20))  # Output: 30
```

### Exercise 4: Last Character of String
```python
def last_char(string):
    return string[-1]

print(last_char("Hello"))  # Output: o

# Lambda version
last_char_lambda = lambda s: s[-1]
print(last_char_lambda("Hello"))  # Output: o
```

---

## Key Takeaways

### Function Benefits
1. **Code Reusability** - Write once, use many times
2. **Modularity** - Break complex problems into smaller parts
3. **Maintainability** - Easy to update and debug
4. **Readability** - Makes code more organized

### Parameter Types Summary
1. **Regular Parameters** - Must be provided
2. **Default Parameters** - Have default values
3. **Variable-length (*args)** - Accept multiple arguments as tuple
4. **Keyword Arguments (**kwargs)** - Accept multiple keyword arguments as dictionary

### Lambda Functions
- **Anonymous functions** for simple operations
- **Single expression** only
- **Great for** short functions used with map(), filter(), reduce()

### Higher-Order Functions
- **map()** - Transform each element
- **filter()** - Select elements based on condition  
- **reduce()** - Combine all elements into single value

This comprehensive guide provides everything needed to understand and use Python functions effectively, from basic concepts to advanced techniques that even beginners can follow and apply.
