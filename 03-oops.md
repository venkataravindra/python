# Python Object-Oriented Programming (OOP) - Complete Guide

## What is Object-Oriented Programming?

**Object-Oriented Programming (OOP)** is a programming approach that organizes code into **classes** and **objects**. Think of it as creating blueprints (classes) to build real-world items (objects).

### Real-World Analogy
- **Class** = Blueprint of a house
- **Object** = Actual house built from that blueprint
- **Attributes** = Features of the house (color, size, rooms)
- **Methods** = Actions the house can perform (open door, turn on lights)

---

## Class Basics

### What is a Class?

A **class** is a collection of **variables** (attributes) and **functions** (methods) grouped together.

### Basic Syntax
```python
class ClassName:
    # Class body
    pass  # 'pass' keyword creates an empty class
```

**Key Points:**
- `class` is the keyword used to **declare** a class
- `pass` is used to create an **empty class**
- Class names should follow **PascalCase** convention (e.g., `MyClass`, `StudentRecord`)

---

## Constructors and Objects

### Constructor (`__init__`)

A **constructor** is a special method used to **initialize** instance members when an object is created.

```python
class Test:
    def __init__(self):
        self.num1 = 200
        self.num2 = 100

# Creating objects
obj1 = Test()
obj1.num1 = 2000  # Modifying object's attribute

obj2 = Test()
print(obj2.num1)  # Output: 200 (original value)
```

### Constructor with Parameters

```python
class Test:
    def __init__(self, param1, param2):
        self.num1 = param1
        self.num2 = param2

obj1 = Test(200, 100)
res = obj1.num1 + obj1.num2
print(f"Addition: {res}")  # Output: Addition: 300
```

### Dynamic Attribute Assignment

```python
class Test:
    def __init__(self):
        pass

obj1 = Test()
obj1.x = 200  # Adding attribute dynamically
print(obj1.x)  # Output: 200
```

---

## Methods in Classes

### Types of Methods

#### 1. No Parameters - No Return Type
```python
class Test:
    def square1(self):
        x = 10
        res = x * x
        print(f"Square: {res}")

obj1 = Test()
obj1.square1()  # Output: Square: 100
```

#### 2. With Parameters - No Return Type
```python
class Test:
    def square2(self, num1):
        res = num1 * num1
        print(f"Square: {res}")

obj1 = Test()
obj1.square2(5)  # Output: Square: 25
```

#### 3. No Parameters - With Return Type
```python
class Test:
    def square3(self):
        x = 10
        res = x * x
        return res

obj1 = Test()
result = obj1.square3()
print(result)  # Output: 100
```

#### 4. With Parameters - With Return Type
```python
class Test:
    def square4(self, num1):
        res = num1 * num1
        return res

obj1 = Test()
result = obj1.square4(7)
print(result)  # Output: 49
```

---

## Inheritance

**Inheritance** allows getting data from a **parent class** to a **child class**. It promotes code reusability.

### Types of Inheritance

#### 1. Single Inheritance
One child class inherits from one parent class.

```python
class Parent:
    def __init__(self):
        self.msg = "Hello"

class Child(Parent):
    def __init__(self):
        super().__init__()  # Call parent constructor
        self.sub = "Agentic AI"

obj1 = Child()
print(obj1.msg)  # Output: Hello (from parent)
print(obj1.sub)  # Output: Agentic AI (from child)

obj2 = Parent()
print(obj2.msg)  # Output: Hello
# print(obj2.sub)  # This would cause an error
```

#### 2. Single Inheritance with Parameters
```python
class Parent:
    def __init__(self, param1):
        self.msg = param1

class Child(Parent):
    def __init__(self, param1, param2):
        super().__init__(param1)  # Pass parameter to parent
        self.sub = param2

obj1 = Child("Hello", "Agentic AI")
print(obj1.msg)  # Output: Hello
print(obj1.sub)  # Output: Agentic AI
```

#### 3. Multi-Level Inheritance
Child class inherits from parent, and another class inherits from child.

```python
class Parent:
    def __init__(self):
        self.num1 = 300

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.num2 = 200

class SubChild(Child):
    def __init__(self):
        super().__init__()
        self.num3 = 100

obj1 = SubChild()
print(obj1.num1 + obj1.num2 + obj1.num3)  # Output: 600
```

#### 4. Multiple Inheritance
One child class inherits from multiple parent classes.

```python
class Parent1:
    def __init__(self):
        self.num1 = 200

class Parent2:
    def __init__(self):
        self.num2 = 100

class Child(Parent1, Parent2):
    def __init__(self):
        Parent1.__init__(self)
        Parent2.__init__(self)

obj = Child()
print(f"Addition: {obj.num1 + obj.num2}")  # Output: Addition: 300
```

#### 5. Multiple Inheritance with Conflicts
```python
class Parent1:
    def __init__(self):
        self.num1 = 200

class Parent2:
    def __init__(self):
        self.num1 = 2000  # Same attribute name

class Child(Parent1, Parent2):
    def __init__(self):
        Parent1.__init__(self)
        Parent2.__init__(self)  # This overwrites Parent1's value

obj = Child()
print(obj.num1)  # Output: 2000 (last assignment wins)
```

#### 6. Hierarchical Inheritance
Multiple child classes inherit from one parent class.

```python
class Parent:
    def __init__(self):
        self.x = 1000

class Child1(Parent):
    def __init__(self):
        super().__init__()
        self.y = 2000

class Child2(Parent):
    def __init__(self):
        super().__init__()
        self.y = 20000

obj1 = Child1()
print(obj1.x, obj1.y)  # Output: 1000 2000

obj2 = Child2()
print(obj2.x, obj2.y)  # Output: 1000 20000
```

---

## Access Modifiers

### 1. Public Variables
Default access level - accessible everywhere.

```python
class Test:
    def __init__(self):
        self.x = 100  # Public variable

obj = Test()
print(obj.x)  # Output: 100 (accessible)
```

### 2. Private Variables (`__`)
Cannot be accessed outside the class or in child classes.

```python
class Test:
    def __init__(self):
        self.__x = 200  # Private variable

obj = Test()
# print(obj.__x)  # This causes AttributeError
```

#### Accessing Private Variables
```python
class Test:
    def __init__(self):
        self.__x = 100

    def setX(self, value):
        self.__x = value
    
    def getX(self):
        return self.__x

obj = Test()
print(obj.getX())    # Output: 100
obj.setX(1000)
print(obj.getX())    # Output: 1000
```

#### Private Variables in Inheritance
```python
class Parent:
    def __init__(self):
        self.__x = 200  # Private - not inherited

class Child(Parent):
    pass

obj = Child()
# print(obj.__x)  # This causes AttributeError
```

### 3. Protected Variables (`_`)
Accessible within the class and child classes.

```python
class Parent:
    def __init__(self):
        self._x = 100  # Protected variable

class Child(Parent):
    pass

obj = Child()
print(obj._x)  # Output: 100 (accessible in child class)
```

---

## Class vs Instance Variables

### Instance Variables
Belong to individual objects.

```python
class Test:
    def __init__(self):
        self.name = "STD1"  # Instance variable

obj1 = Test()
obj2 = Test()
obj1.name = "STD2"
print(obj1.name)  # Output: STD2
print(obj2.name)  # Output: STD1
```

### Class Variables
Shared among all instances of the class.

```python
class Test:
    name = "JNTU"  # Class variable

obj1 = Test()
obj2 = Test()
print(obj1.name)  # Output: JNTU
print(obj2.name)  # Output: JNTU

# Modifying class variable
Test.name = "KLU"
print(obj1.name)  # Output: KLU
print(obj2.name)  # Output: KLU
```

### Class Methods (`@classmethod`)
```python
class Test:
    name = "JNTU"

    @classmethod
    def test_func(cls):
        cls.name = "KLU"  # Modify class variable

Test.test_func()
print(Test.name)  # Output: KLU
```

---

## Method Overriding (Polymorphism)

Child class can override parent class methods.

```python
class Parent:
    def dbfunc(self):
        return "Oracle DB Connection Soon..!"

class Child(Parent):
    def dbfunc(self):  # Override parent method
        return "VectorDB Connection Soon..!"

obj = Child()
result = obj.dbfunc()
print(result)  # Output: VectorDB Connection Soon..!
```

---

## Method Overloading

Python doesn't support traditional method overloading, but you can simulate it.

### Problem with Multiple Methods
```python
class Test:
    def test(self, x):
        print(x)
    
    def test(self, x, y):  # This overwrites the previous method
        print(x, y)

obj = Test()
# obj.test(100)      # This causes TypeError
obj.test(100, 200)   # Output: 100 200
```

### Solution: Variable Arguments
```python
class Test:
    def test(self, *param):
        print(param)

obj = Test()
obj.test(10)              # Output: (10,)
obj.test(10, 20, 30, 40, 50)  # Output: (10, 20, 30, 40, 50)
```

---

## Static Methods

Utility methods that don't need class or instance data.

```python
class Calculator:
    @staticmethod
    def add(x, y):
        return x + y

# Call without creating object
result = Calculator.add(200, 100)
print(result)  # Output: 300
```

---

## Abstract Classes and Methods

Force child classes to implement specific methods.

```python
from abc import ABC, abstractmethod

class Test(ABC):
    @abstractmethod
    def start_business(self):
        pass

class Child(Test):
    def start_business(self):  # Must implement this method
        return "Start EduTech / Software Solutions"

obj = Child()
print(obj.start_business())  # Output: Start EduTech / Software Solutions
```

---

## Special Methods

### `__str__` Method
Defines string representation of object.

```python
class Test:
    def __str__(self):
        return "VPro"

obj = Test()
print(obj)  # Output: VPro
```

### Constructor Overriding
```python
class Test:
    def __init__(self):
        self.x = 100
    
    def __init__(self, name):  # This overwrites the previous constructor
        self.name = name

obj = Test("VPro")  # Must provide name parameter
```

---

## Key Concepts Summary

### Important Keywords
- **`self`** - Refers to the current instance
- **`super()`** - Calls parent class methods
- **`cls`** - Refers to the class (used in class methods)
- **`@staticmethod`** - Creates utility methods
- **`@classmethod`** - Creates class-level methods
- **`@abstractmethod`** - Creates abstract methods

### Access Levels
- **Public** - `variable` (accessible everywhere)
- **Protected** - `_variable` (accessible in class and subclasses)
- **Private** - `__variable` (accessible only within the class)

### Inheritance Benefits
1. **Code Reusability** - Don't repeat code
2. **Extensibility** - Add new features to existing classes
3. **Maintainability** - Changes in parent affect all children
4. **Polymorphism** - Same interface, different implementations

### Best Practices
1. Use **meaningful class names** (PascalCase)
2. Keep **constructors simple**
3. Use **inheritance** for "is-a" relationships
4. Use **composition** for "has-a" relationships
5. Follow **single responsibility principle**
6. Use **abstract classes** for common interfaces

This comprehensive guide covers all essential OOP concepts in Python with practical examples that demonstrate real-world usage patterns.
