# Python Dictionaries - Complete Reference Guide

## Table of Contents
1. [Introduction to Dictionaries](#introduction)
2. [Dictionary Characteristics](#characteristics)
3. [Examples with Explanations](#examples)
4. [Dictionary Methods Reference](#methods)
5. [Common Patterns and Use Cases](#patterns)
6. [Best Practices](#best-practices)

---

## Introduction to Dictionaries {#introduction}

Dictionaries are one of the most important and versatile data types in Python. They store data in key-value pairs, providing fast lookup and flexible data organization.

## Dictionary Characteristics {#characteristics}

1. **Key-Value Pairs**: Data stored as associated pairs
2. **Keys are Immutable**: Keys must be hashable (strings, numbers, tuples)
3. **Values are Mutable**: Values can be any data type and can be changed
4. **Ordered**: Maintains insertion order (Python 3.7+)
5. **No Duplicates**: Keys must be unique
6. **Syntax**: Uses curly braces `{}` or `dict()` function
7. **Fast Lookup**: O(1) average time complexity for access

---

## Examples with Explanations {#examples}

### Example 1: Basic Dictionary Creation and Access

```python
# Creating a dictionary
d1 = {
    "id": 101,
    "name": "AgenticAI",
    "version": 2.0
}

print(d1)                    # {'id': 101, 'name': 'AgenticAI', 'version': 2.0}
print(d1.keys())             # dict_keys(['id', 'name', 'version'])
print(d1.values())           # dict_values([101, 'AgenticAI', 2.0])
print(d1.items())            # dict_items([('id', 101), ('name', 'AgenticAI'), ('version', 2.0)])
```

**Key Learning**: 
- `keys()`: Returns all keys as a dict_keys object
- `values()`: Returns all values as a dict_values object  
- `items()`: Returns key-value pairs as tuples in a dict_items object

### Example 2: Alternative Dictionary Creation

```python
# Using dict() constructor
d1 = dict({"id": 1, "name": "test"})
print(d1)                    # {'id': 1, 'name': 'test'}
print(type(d1))              # <class 'dict'>

# Alternative syntax
d2 = dict(id=1, name="test")  # Keyword arguments
print(d2)                    # {'id': 1, 'name': 'test'}
```

**Key Learning**: Multiple ways to create dictionaries using `dict()` constructor.

### Example 3: Duplicate Keys Behavior

```python
d1 = {
    "name": "Test1",
    "name": 123              # This overwrites the previous value
}
print(d1)                    # {'name': 123}
```

**Key Learning**: Duplicate keys are not allowed; the last value overwrites previous ones.

### Example 4: Different Data Types as Values

```python
d1 = {
    "id": 101,
    "name": 101,             # Same value, different meaning
}
print(d1)                    # {'id': 101, 'name': 101}
```

**Key Learning**: Values can be of any type, including the same value for different keys.

### Example 5: Dictionary Operations and Error Handling

```python
d1 = {}

# Adding and modifying elements
d1["key1"] = 100
d1["key2"] = 200
d1["key1"] = 1000            # Overwrites existing value
d1["key3"] = 300

# Accessing elements
print(d1["key1"])            # 1000 (direct access)
print(d1.get("key1"))        # 1000 (safe access)

# Error handling
# print(d1["key4"])          # KeyError: 'key4'
print(d1.get("key4"))        # None (no error)
print(d1.get("key4", 0))     # 0 (default value)

# Removing elements
del d1["key3"]               # Removes key3
# del d1["key4"]             # KeyError if key doesn't exist
removed_value = d1.pop("key2")  # Removes and returns value
print(d1)                    # {'key1': 1000}
```

**Key Learning**:
- `dict[key]`: Direct access, raises KeyError if key doesn't exist
- `dict.get(key)`: Safe access, returns None if key doesn't exist
- `dict.get(key, default)`: Returns default value if key doesn't exist
- `del dict[key]`: Removes key-value pair
- `dict.pop(key)`: Removes and returns value

### Example 6: Dictionary Iteration

```python
d1 = {
    "key1": 100,
    "key2": 200,
    "key3": 300
}

# Iterate over keys (default behavior)
for key in d1:
    print(key)               # key1, key2, key3

# Iterate over values
for value in d1.values():
    print(value)             # 100, 200, 300

# Iterate over key-value pairs
for key, value in d1.items():
    print(key, value)        # key1 100, key2 200, key3 300
```

**Key Learning**: Three ways to iterate through dictionaries for different use cases.

### Example 7: Nested Dictionaries

```python
d1 = {
    101: {
        "key1": 100
    },
    102: {
        "key1": 200
    }
}

# Accessing nested values
print(d1.get(101).get("key1"))    # 100 (safe chaining)
print(d1[102]["key1"])            # 200 (direct access)

# Iterating through nested dictionaries
for inner_d1 in d1.values():
    print(inner_d1.get("key1"))   # 100, 200
```

**Key Learning**: Dictionaries can contain other dictionaries; use chaining for nested access.

### Example 8: Dictionary Comprehension

```python
# Creating dictionary using comprehension
d1 = {x: x*x for x in range(1, 6)}
print(d1)                    # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

**Key Learning**: Dictionary comprehensions provide concise way to create dictionaries.

### Example 9: Character Frequency Counter

```python
msg = "hello"
count = {}

for ch in msg:
    count[ch] = count.get(ch, 0) + 1

print(count)                 # {'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

**Key Learning**: Common pattern for counting occurrences using `get()` with default value.

### Example 10: Word Frequency Counter

```python
statement = "python java python java python"
words = statement.split()
count = {}

for word in words:
    count[word] = count.get(word, 0) + 1

print(count)                 # {'python': 3, 'java': 2}
```

**Key Learning**: Extending the counting pattern for word frequency analysis.

### Example 11: Dictionary Merging

```python
d1 = {"id": 101}
d2 = {"name": "VPro"}

d1.update(d2)                # Merges d2 into d1
print(d1)                    # {'id': 101, 'name': 'VPro'}
```

**Key Learning**: `update()` method merges one dictionary into another.

### Example 12: Key Existence Checking

```python
d1 = {
    "id": 101,
    "name": "VPro",
    "version": 2.0
}

if "name" in d1:
    print("Existed")         # This will print
else:
    print("Not Existed")
```

**Key Learning**: Use `in` operator for efficient key existence checking.

### Example 13: Finding Maximum Value

```python
d1 = {
    "std1": 50,
    "std2": 60,
    "std3": 70,
    "std4": 80
}

top_student = max(d1, key=d1.get)  # Find key with maximum value
print(top_student)           # std4
print(d1[top_student])       # 80
```

**Key Learning**: `max()` with `key=dict.get` finds the key with the maximum value.

### Example 14: Dictionary Sorting

```python
d1 = {
    "Rahul": 88,
    "Samba": 95,
    "Anil": 91
}

# Sort by keys
d2 = dict(sorted(d1.items()))
print(d2)                    # {'Anil': 91, 'Rahul': 88, 'Samba': 95}

# Sort by values
d3 = dict(sorted(d1.items(), key=lambda item: item[1]))
print(d3)                    # {'Rahul': 88, 'Anil': 91, 'Samba': 95}
```

**Key Learning**: 
- `sorted(dict.items())`: Sorts by keys
- `sorted(dict.items(), key=lambda item: item[1])`: Sorts by values

### Example 15: Sum of Dictionary Values

```python
d1 = {
    "key1": 1,
    "key2": 2,
    "key3": 3    
}

res = sum(d1.values())
print(res)                   # 6
```

**Key Learning**: `sum()` can be applied to dictionary values for numeric data.

### Example 16: Dictionary Inversion

```python
d1 = {
    "101": "Hello",                
    "102": "Agentic AI"
}

# Swap keys and values
d2 = {value: key for key, value in d1.items()}
print(d2)                    # {'Hello': '101', 'Agentic AI': '102'}
```

**Key Learning**: Dictionary comprehension can invert key-value relationships.

### Example 17: Remove Duplicate Values

```python
d1 = {
    "A": 10,
    "B": 20,
    "C": 10,
    "D": 30,
    "E": 20
}

result = {}
for key, value in d1.items():
    if value not in result.values():
        result[key] = value

print(result)                # {'A': 10, 'B': 20, 'D': 30}
```

**Key Learning**: Algorithm to keep only first occurrence of each unique value.

### Example 18: Find Common Keys

```python
d1 = {
    "A": 10,
    "B": 20
}
d2 = {
    "B": 40,
    "C": 50
}

common = d1.keys() & d2.keys()  # Set intersection
print(common)                # {'B'}
```

**Key Learning**: Dictionary keys support set operations for finding commonalities.

---

## Dictionary Methods Reference {#methods}

### Access Methods

| Method | Description | Example | Returns |
|--------|-------------|---------|---------|
| `dict[key]` | Direct access to value | `d["name"]` | Value or KeyError |
| `get(key)` | Safe access to value | `d.get("name")` | Value or None |
| `get(key, default)` | Safe access with default | `d.get("name", "Unknown")` | Value or default |

### Modification Methods

| Method | Description | Example | Notes |
|--------|-------------|---------|-------|
| `dict[key] = value` | Add/update key-value pair | `d["name"] = "John"` | Creates or updates |
| `update(other)` | Merge another dict | `d1.update(d2)` | Overwrites existing keys |
| `setdefault(key, default)` | Get or set default value | `d.setdefault("count", 0)` | Returns existing or sets default |

### Removal Methods

| Method | Description | Example | Returns |
|--------|-------------|---------|---------|
| `del dict[key]` | Remove key-value pair | `del d["name"]` | None (raises KeyError if not found) |
| `pop(key)` | Remove and return value | `value = d.pop("name")` | Removed value (raises KeyError if not found) |
| `pop(key, default)` | Remove and return with default | `value = d.pop("name", None)` | Removed value or default |
| `popitem()` | Remove and return last item | `key, value = d.popitem()` | (key, value) tuple |
| `clear()` | Remove all items | `d.clear()` | None |

### View Methods

| Method | Description | Example | Returns |
|--------|-------------|---------|---------|
| `keys()` | Get all keys | `d.keys()` | dict_keys object |
| `values()` | Get all values | `d.values()` | dict_values object |
| `items()` | Get all key-value pairs | `d.items()` | dict_items object |

### Utility Methods

| Method | Description | Example | Returns |
|--------|-------------|---------|---------|
| `copy()` | Create shallow copy | `d2 = d1.copy()` | New dictionary |
| `fromkeys(keys, value)` | Create dict from keys | `dict.fromkeys(['a','b'], 0)` | New dictionary |

---

## Common Patterns and Use Cases {#patterns}

### 1. Counting Pattern

```python
# Count occurrences
def count_items(items):
    counts = {}
    for item in items:
        counts[item] = counts.get(item, 0) + 1
    return counts

# Using defaultdict (alternative)
from collections import defaultdict
counts = defaultdict(int)
for item in items:
    counts[item] += 1
```

### 2. Grouping Pattern

```python
# Group items by some criteria
def group_by_length(words):
    groups = {}
    for word in words:
        length = len(word)
        if length not in groups:
            groups[length] = []
        groups[length].append(word)
    return groups
```

### 3. Caching Pattern

```python
# Simple memoization
cache = {}
def expensive_function(n):
    if n in cache:
        return cache[n]
    
    result = complex_calculation(n)
    cache[n] = result
    return result
```

### 4. Configuration Pattern

```python
# Application configuration
config = {
    "database": {
        "host": "localhost",
        "port": 5432,
        "name": "myapp"
    },
    "api": {
        "timeout": 30,
        "retries": 3
    }
}

# Access nested config
db_host = config["database"]["host"]
```

### 5. Data Transformation Pattern

```python
# Transform list of dicts
students = [
    {"name": "Alice", "grade": 85},
    {"name": "Bob", "grade": 92}
]

# Create name to grade mapping
grades = {student["name"]: student["grade"] for student in students}
```

---

## Best Practices {#best-practices}

### 1. Use `get()` for Safe Access

```python
# Good
name = user.get("name", "Unknown")

# Avoid
try:
    name = user["name"]
except KeyError:
    name = "Unknown"
```

### 2. Use `setdefault()` for Default Values

```python
# Good
groups.setdefault(key, []).append(item)

# Avoid
if key not in groups:
    groups[key] = []
groups[key].append(item)
```

### 3. Use Dictionary Comprehensions

```python
# Good
squares = {x: x**2 for x in range(10)}

# Less readable
squares = {}
for x in range(10):
    squares[x] = x**2
```
### 4. Use `items()` for Key-Value Iteration

```python
# Good
for key, value in dictionary.items():
    print(f"{key}: {value}")

# Avoid
for key in dictionary:
    value = dictionary[key]
    print(f"{key}: {value}")
```

### 5. Use `in` for Key Existence

```python
# Good
if "key" in dictionary:
    process(dictionary["key"])

# Avoid
if dictionary.get("key") is not None:
    process(dictionary["key"])
```

### 6. Use `update()` for Merging

```python
# Good
config.update(user_settings)

# Avoid
for key, value in user_settings.items():
    config[key] = value
```

### 7. Use Meaningful Key Names

```python
# Good
user_data = {
    "user_id": 123,
    "username": "john_doe",
    "email": "john@example.com",
    "created_at": "2024-01-01"
}

# Avoid
data = {
    "id": 123,
    "name": "john_doe",
    "mail": "john@example.com",
    "date": "2024-01-01"
}
```

### 8. Handle Missing Keys Gracefully

```python
# Good
def get_user_info(user_dict):
    return {
        "name": user_dict.get("name", "Unknown"),
        "age": user_dict.get("age", 0),
        "email": user_dict.get("email", "No email provided")
    }

# Avoid
def get_user_info(user_dict):
    try:
        name = user_dict["name"]
    except KeyError:
        name = "Unknown"
    # ... repeat for each key
```

---

## Advanced Dictionary Techniques

### 1. Nested Dictionary Access

```python
def safe_get_nested(dictionary, *keys, default=None):
    """Safely access nested dictionary values"""
    for key in keys:
        if isinstance(dictionary, dict) and key in dictionary:
            dictionary = dictionary[key]
        else:
            return default
    return dictionary

# Usage
data = {"user": {"profile": {"name": "John"}}}
name = safe_get_nested(data, "user", "profile", "name", default="Unknown")
```

### 2. Dictionary Flattening

```python
def flatten_dict(d, parent_key='', sep='_'):
    """Flatten nested dictionary"""
    items = []
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_dict(v, new_key, sep=sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

# Example
nested = {
    "user": {
        "name": "John",
        "address": {
            "city": "New York",
            "zip": "10001"
        }
    }
}
flat = flatten_dict(nested)
# Result: {'user_name': 'John', 'user_address_city': 'New York', 'user_address_zip': '10001'}
```

### 3. Dictionary Filtering

```python
# Filter by keys
def filter_dict_by_keys(dictionary, allowed_keys):
    return {k: v for k, v in dictionary.items() if k in allowed_keys}

# Filter by values
def filter_dict_by_values(dictionary, condition):
    return {k: v for k, v in dictionary.items() if condition(v)}

# Examples
data = {"a": 1, "b": 2, "c": 3, "d": 4}
filtered_keys = filter_dict_by_keys(data, ["a", "c"])  # {"a": 1, "c": 3}
filtered_values = filter_dict_by_values(data, lambda x: x > 2)  # {"c": 3, "d": 4}
```

### 4. Dictionary Merging Strategies

```python
# Simple merge (overwrites)
def merge_dicts(*dicts):
    result = {}
    for d in dicts:
        result.update(d)
    return result

# Deep merge for nested dictionaries
def deep_merge(dict1, dict2):
    result = dict1.copy()
    for key, value in dict2.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = deep_merge(result[key], value)
        else:
            result[key] = value
    return result

# Python 3.9+ merge operator
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}
merged = dict1 | dict2  # {"a": 1, "b": 3, "c": 4}
```

---

## Performance Considerations

### Time Complexity

| Operation | Average Case | Worst Case | Notes |
|-----------|--------------|------------|-------|
| Access `dict[key]` | O(1) | O(n) | Hash collision worst case |
| Insert `dict[key] = value` | O(1) | O(n) | May trigger resize |
| Delete `del dict[key]` | O(1) | O(n) | Hash collision worst case |
| Search `key in dict` | O(1) | O(n) | Hash collision worst case |
| Iteration | O(n) | O(n) | Must visit all items |

### Memory Considerations

```python
# Memory-efficient dictionary creation
# Good for large datasets
import sys

# Using __slots__ for dictionary-like objects
class Point:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Using dict for flexibility vs memory trade-off
points_dict = [{"x": i, "y": i*2} for i in range(1000)]  # More memory
points_slots = [Point(i, i*2) for i in range(1000)]      # Less memory
```

---

## Common Pitfalls and Solutions

### 1. Mutable Default Arguments

```python
# Problem
def add_item(item, target_dict={}):  # Dangerous!
    target_dict[item] = True
    return target_dict

# Solution
def add_item(item, target_dict=None):
    if target_dict is None:
        target_dict = {}
    target_dict[item] = True
    return target_dict
```

### 2. Modifying Dictionary During Iteration

```python
# Problem
d = {"a": 1, "b": 2, "c": 3}
for key in d:
    if d[key] > 1:
        del d[key]  # RuntimeError!

# Solution
d = {"a": 1, "b": 2, "c": 3}
keys_to_delete = [key for key, value in d.items() if value > 1]
for key in keys_to_delete:
    del d[key]
```

### 3. Shallow vs Deep Copy

```python
import copy

original = {"a": [1, 2, 3], "b": [4, 5, 6]}

# Shallow copy - nested objects are shared
shallow = original.copy()
shallow["a"].append(4)  # Modifies original too!

# Deep copy - completely independent
deep = copy.deepcopy(original)
deep["a"].append(4)  # Original unchanged
```

---

## Real-World Examples

### 1. Configuration Management

```python
class Config:
    def __init__(self):
        self._config = {
            "database": {
                "host": "localhost",
                "port": 5432,
                "timeout": 30
            },
            "logging": {
                "level": "INFO",
                "format": "%(asctime)s - %(levelname)s - %(message)s"
            }
        }
    
    def get(self, path, default=None):
        """Get config value using dot notation"""
        keys = path.split('.')
        value = self._config
        for key in keys:
            if isinstance(value, dict) and key in value:
                value = value[key]
            else:
                return default
        return value
    
    def set(self, path, value):
        """Set config value using dot notation"""
        keys = path.split('.')
        config = self._config
        for key in keys[:-1]:
            config = config.setdefault(key, {})
        config[keys[-1]] = value

# Usage
config = Config()
db_host = config.get("database.host")  # "localhost"
config.set("database.host", "production-server")
```

### 2. Data Processing Pipeline

```python
def process_sales_data(sales_records):
    """Process sales data and generate summary statistics"""
    
    # Group by product
    product_sales = {}
    for record in sales_records:
        product = record["product"]
        amount = record["amount"]
        
        if product not in product_sales:
            product_sales[product] = {
                "total_sales": 0,
                "count": 0,
                "amounts": []
            }
        
        product_sales[product]["total_sales"] += amount
        product_sales[product]["count"] += 1
        product_sales[product]["amounts"].append(amount)
    
    # Calculate statistics
    summary = {}
    for product, data in product_sales.items():
        amounts = data["amounts"]
        summary[product] = {
            "total_sales": data["total_sales"],
            "average_sale": data["total_sales"] / data["count"],
            "min_sale": min(amounts),
            "max_sale": max(amounts),
            "transaction_count": data["count"]
        }
    
    return summary

# Usage
sales = [
    {"product": "laptop", "amount": 1200},
    {"product": "mouse", "amount": 25},
    {"product": "laptop", "amount": 1500},
    {"product": "mouse", "amount": 30}
]

summary = process_sales_data(sales)
```

### 3. Caching Decorator

```python
def memoize(func):
    """Decorator to cache function results"""
    cache = {}
    
    def wrapper(*args, **kwargs):
        # Create cache key from arguments
        key = str(args) + str(sorted(kwargs.items()))
        
        if key not in cache:
            cache[key] = func(*args, **kwargs)
            print(f"Cached result for {func.__name__}{args}")
        else:
            print(f"Retrieved from cache for {func.__name__}{args}")
        
        return cache[key]
    
    wrapper.cache = cache  # Expose cache for inspection
    return wrapper

# Usage
@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

result = fibonacci(10)  # Calculates and caches intermediate results
```

---

## Testing Dictionary Code

### Unit Testing Examples

```python
import unittest

class TestDictionaryOperations(unittest.TestCase):
    
    def setUp(self):
        self.sample_dict = {"a": 1, "b": 2, "c": 3}
    
    def test_key_access(self):
        self.assertEqual(self.sample_dict["a"], 1)
        self.assertEqual(self.sample_dict.get("d"), None)
        self.assertEqual(self.sample_dict.get("d", 0), 0)
    
    def test_key_modification(self):
        self.sample_dict["d"] = 4
        self.assertIn("d", self.sample_dict)
        self.assertEqual(self.sample_dict["d"], 4)
    
    def test_key_deletion(self):
        del self.sample_dict["a"]
        self.assertNotIn("a", self.sample_dict)
        
        with self.assertRaises(KeyError):
            del self.sample_dict["nonexistent"]
    
    def test_dictionary_methods(self):
        keys = list(self.sample_dict.keys())
        values = list(self.sample_dict.values())
        items = list(self.sample_dict.items())
        
        self.assertEqual(len(keys), 3)
        self.assertEqual(len(values), 3)
        self.assertEqual(len(items), 3)

if __name__ == "__main__":
    unittest.main()
```

---

## Summary

Python dictionaries are powerful, flexible data structures that provide:

- **Fast Access**: O(1) average time complexity for most operations
- **Flexible Storage**: Can store any combination of data types
- **Rich Methods**: Comprehensive set of built-in methods for manipulation
- **Memory Efficient**: Optimized internal implementation
- **Versatile**: Suitable for caching, configuration, data processing, and more

### Key Takeaways:

1. Use `get()` for safe access to avoid KeyError exceptions
2. Leverage dictionary comprehensions for concise code
3. Use `items()` when you need both keys and values
4. Be careful with mutable default arguments
5. Consider memory and performance implications for large datasets
6. Use meaningful key names for better code readability
7. Handle missing keys gracefully in production code

Dictionaries are fundamental to Python programming and mastering their usage patterns will significantly improve your code quality and efficiency.
