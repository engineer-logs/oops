## Class

A class is a **blueprint or template**. It defines what attributes (data) and methods (actions) every object of that type will have. A class by itself takes up **no memory** — it's just a definition until an object is created from it.

```python
class Employee:
    def __init__(self):
        self.__salary = 0        # private attribute (name-mangled with __)
        self.employeeName = ""   # public attribute

    def setName(self, name):
        self.employeeName = name

    def setSalary(self, val):
        self.__salary = val

    def getSalary(self):
        return self.__salary
```

Two things to notice here: `employeeName` is public — anyone can access it directly. `__salary` is private — the double underscore hides it from outside access. You must go through `getSalary()` to read it. This is encapsulation in action.

---

## Object

An object is an **instance of a class** — the real thing built from the blueprint. When you create an object, Python allocates memory for it. Each object gets its own separate copy of the data and cannot touch another object's data.

```python
# Creating objects from the Employee class
obj1 = Employee()
obj1.setName("Aju")
obj1.setSalary(10000)

obj2 = Employee()
obj2.setName("Hari")
obj2.setSalary(15000)

# Accessing data
print("Salary of " + obj1.employeeName + " is " + str(obj1.getSalary()))
print("Salary of " + obj2.employeeName + " is " + str(obj2.getSalary()))
```

Output:
```
Salary of Aju is 10000
Salary of Hari is 15000
```

`obj1` and `obj2` share the same class structure but hold completely independent data. Changing Aju's salary has zero effect on Hari's.

Here's what this looks like in memory:
<img width="1440" height="656" alt="image" src="https://github.com/user-attachments/assets/f884848e-9230-4cb8-9782-07ffe959926e" />


## Attributes vs Behaviours

**Attributes** are the data — they describe the state of an object. In `Employee`, `employeeName` and `__salary` are attributes.

**Methods (Behaviours)** are the actions — what the object can do. In `Employee`, `setName()`, `setSalary()`, and `getSalary()` are methods.

A simple way to remember: attributes are *nouns* (what it has), methods are *verbs* (what it does).

---

## Public vs Private Attributes

```python
obj1 = Employee()
obj1.setName("Aju")

# Public — works fine, direct access allowed
print(obj1.employeeName)     # Output: Aju

# Private — this will throw an AttributeError
print(obj1.__salary)         # ERROR! Can't access directly

# Correct way — use the method
print(obj1.getSalary())      # Output: 10000
```

In Python, private attributes use double underscore (`__`) prefix. Python internally renames them to `_ClassName__attribute` (called name mangling), making them harder to access from outside. This protects sensitive data like salary or passwords.

---

## Creating and Deleting Objects

```python
# Creating an object — memory gets allocated on the heap
obj1 = Employee()

# Deleting an object — removes the reference
del obj1

# After del, obj1 no longer points to anything
# Python's Garbage Collector will free the memory automatically
```

You don't need to manually free memory in Python. The GC handles it once no variable is pointing to the object.

---

## Stack and Heap Memory

```python
obj1 = Employee()   # 'obj1' label lives on the stack
                    # actual Employee data lives on the heap
obj2 = Employee()   # same — separate heap block for obj2
```
<img width="1440" height="572" alt="image" src="https://github.com/user-attachments/assets/1b5620fd-8019-4448-b0f4-eb0462432482" />

In Python, the variable names (`obj1`, `obj2`) live on the stack as simple references. The actual object data — all the attributes — lives on the heap. When you do `del obj1`, only the reference is removed. The heap memory is cleaned up automatically by Python's Garbage Collector once nothing is pointing to it.

---

## Quick Recall Cheat Sheet

| Concept | One-line summary | Python syntax |
|---|---|---|
| Class | Blueprint, no memory | `class Employee:` |
| Object | Instance with own memory | `obj1 = Employee()` |
| `__init__` | Constructor — runs on creation | `def __init__(self):` |
| Public attribute | Accessible directly | `self.employeeName` |
| Private attribute | Hidden, use methods | `self.__salary` |
| Method | Action the object performs | `def getSalary(self):` |
| Delete object | Remove reference | `del obj1` |
| Garbage Collector | Auto-frees unused heap memory | handled by Python |
