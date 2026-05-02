# Class vs Instance Variables in Python

> **Object-Oriented Programming · Python**

---

## Overview

In Python, variables defined inside a class can be either **class variables** (shared) or **instance variables** (per-object).

Unlike Java or C++, Python has **no `static` keyword**. Any variable assigned directly inside the class body (outside methods) automatically becomes a class variable.

| Feature | Class Variable | Instance Variable |
|---|---|---|
| Defined in | Class body (outside methods) | `__init__` via `self` |
| Scope | All instances | Single instance only |
| Memory | Allocated **once** per class | Allocated per object |
| Modify correctly | `ClassName.var = value` ✅ | `self.var = value` ✅ |
| Modify via instance | Creates shadow instance var ⚠️ | Works as expected |

---

## Memory Model

![Memory Model](assets/memory_model.svg)

- `school` lives once in the class — both `s1` and `s2` point to the same variable.
- `name` lives separately inside each object.

---

## 1. Class Variables

A class variable belongs to the **class itself**, not to any object. All instances share it.

- Defined inside the class but **outside all methods**
- Accessed using the class name or any object
- Memory allocated **only once**

```python
class Student:
    school = "ABC School"   # class variable

s1 = Student()
s2 = Student()

print(s1.school)   # ABC School
print(s2.school)   # ABC School
```

**Output:**
```
ABC School
ABC School
```

Both `s1` and `s2` read the same variable. The variable belongs to the class, not to individual objects.

---

## 2. Instance Variables

An instance variable belongs to a **specific object**. Each object maintains its own copy.

- Defined inside methods using `self` (usually in `__init__`)
- Unique for each object
- Changing one object's value does not affect others

```python
class Student:
    def __init__(self, name):
        self.name = name   # instance variable

s1 = Student("Aju")
s2 = Student("Hari")

print(s1.name)   # Aju
print(s2.name)   # Hari
```

**Output:**
```
Aju
Hari
```

---

## 3. Both Together — A Realistic Example

```python
class CSStudent:
    stream = 'cse'          # class variable → shared by all

    def __init__(self, name, roll):
        self.name = name    # instance variable
        self.roll = roll    # instance variable

a = CSStudent('Aju', 1)
b = CSStudent('Hari', 2)

print(a.stream)   # cse
print(b.stream)   # cse   ← same variable
print(a.name)     # Aju
print(b.name)     # Hari  ← different values
```

**Output:**
```
cse
cse
Aju
Hari
```

- `stream` is shared by all objects.
- `name` and `roll` are unique to each object.

---

## 4. Modifying Class Variables — Critical Pitfall

There are two ways to modify a class variable. Only one actually modifies it.

### Via an Instance (⚠️ Not Recommended)

```python
a.stream = 'ece'
print(a.stream)   # ece
print(b.stream)   # cse  ← class variable unchanged!
```

**Output:**
```
ece
cse
```

> ⚠️ **Pitfall:** Python does NOT update the class variable. It silently creates a **new instance variable** on `a` that shadows the class variable. Object `b` still reads the original class variable.

![Shadowing Diagram](assets/shadowing_diagram.svg)

### Via the Class Name (✅ Recommended)

```python
CSStudent.stream = 'mech'
print(a.stream)   # ece   ← a has its own instance var, unchanged
print(b.stream)   # mech  ← b sees the updated class variable
```

**Output:**
```
ece
mech
```

> ✅ Always use `ClassName.variable = value` to update a class variable for all instances.

---

## 5. Python Attribute Lookup Order

When you access `obj.attr`, Python searches in this order:

![lookup order](assets/lookup_order.svg)

1. **Instance `__dict__`** — Does the object have its own copy?
2. **Class `__dict__`** — Does the class define it?
3. **Parent classes** — Walk up the inheritance chain (MRO)

This is **why** `a.stream = 'ece'` creates an instance variable — Python writes to the instance dict. Next time, it finds it there first, before the class dict.

```python
a = CSStudent('Aju', 1)
print(a.__dict__)          # {'name': 'Aju', 'roll': 1}
print(CSStudent.__dict__)  # includes 'stream': 'cse'

a.stream = 'ece'
print(a.__dict__)          # {'name': 'Aju', 'roll': 1, 'stream': 'ece'}
print(CSStudent.__dict__)  # 'stream' still 'cse' — class unchanged
```

---

## Interview Questions

**Q1. What is the difference between a class variable and an instance variable?**

A class variable is shared across all objects of the class; an instance variable is unique to each object. Class variables are defined in the class body; instance variables are defined via `self` in `__init__`.

---

**Q2. What happens when you assign to a class variable through an instance?**

Python does not modify the class variable. Instead, it creates a new instance variable on that object, which shadows the class variable. Other instances still see the original class variable.

---

**Q3. How is Python's class variable different from Java's `static` variable?**

Same concept — both are shared across all instances. Python just doesn't need a `static` keyword; any variable assigned at class level becomes a class variable automatically.

---

**Q4. How do you check what instance variables an object has?**

Use `obj.__dict__`. It returns a dictionary of only the instance's own variables. Class variables are stored in `ClassName.__dict__`.

```python
s1 = Student("Aju")
print(s1.__dict__)       # {'name': 'Aju'}
print(Student.__dict__)  # includes 'school': 'ABC School'
```
