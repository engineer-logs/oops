<img width="1440" height="424" alt="image" src="https://github.com/user-attachments/assets/d43f48e3-06e7-4174-9bc7-7e17a730d751" />


## What is a Constructor?

A **constructor** is a special method (`__init__`) that is automatically called when an object is created. Its job is to initialize instance attributes and set the object to a valid starting state.

```python
class Employee:
    def __init__(self):
        self.__salary = 5000          # private attribute
        self.employeeName = "Default" # public attribute

    def setSalary(self, val):
        self.__salary = val

    def getSalary(self):
        return self.__salary

obj = Employee()
print(obj.employeeName)    # Default
print(obj.getSalary())     # 5000
```

**Key points:**
- Always named `__init__`
- `self` is the first parameter — refers to the current instance
- Does not return a value
- If omitted, Python provides a default (empty) constructor automatically

---

## Default Constructor Behavior

If no `__init__` is defined, Python creates the object but sets no instance attributes. Accessing an undefined attribute raises `AttributeError`.

```python
# Case 1: No constructor — no attributes
class Employee:
    pass

e1 = Employee()
# e1.salary  →  AttributeError

# Case 2: Explicit constructor — attributes initialized
class Employee:
    def __init__(self):
        self.id = 0
        self.salary = 0.0
        self.name = None

e2 = Employee()
print(e2.id)      # 0
print(e2.salary)  # 0.0
print(e2.name)    # None
```

**Variable types to remember:**

| Type | Where defined | Scope | Default? |
|---|---|---|---|
| Instance variable | Inside `__init__` via `self` | Per object | No — must assign explicitly |
| Class variable | Inside class, outside methods | Shared by all instances | No |
| Local variable | Inside any method | Method only | No — must init before use |

> **Interview tip:** Python does *not* auto-assign default values to instance variables (unlike Java). If not set in `__init__`, accessing them raises `AttributeError`.

---

## Purpose of a Constructor

1. **Object initialization** — assigns values to attributes at creation time
2. **Code reusability** — initialization logic runs automatically for every object
3. **Valid object state** — ensures the object is ready to use from the start

---

## Types of Constructors


<img width="1440" height="338" alt="image" src="https://github.com/user-attachments/assets/190960d8-da26-4748-a8df-ed7b9ab0e92a" />

### 1. Non-Parameterized Constructor

Takes no arguments other than `self`. Sets fixed default values.

```python
class Employee:
    def __init__(self):
        self.employeeName = "Default"
        self.salary = 0
        print("Employee created!")

emp = Employee()   # Employee created!
```

---

### 2. Parameterized Constructor

Takes extra arguments to initialize the object with custom values at creation time.

```python
class Employee:
    def __init__(self, name, salary):
        self.employeeName = name
        self.salary = salary

obj = Employee("Aju", 10000)
print(obj.employeeName + " earns " + str(obj.salary))
# Aju earns 10000
```

> **`self`** binds the attribute to the specific instance. Python does not require pre-declaring instance variables.

---

### 3. Copy Constructor (Manual)

Python has no built-in copy constructor like C++. Two approaches:

**A) Manual copy using a class method**

```python
import copy

class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        self.skills = []

    @classmethod
    def from_employee(cls, other):
        new_obj = cls(other.name, other.salary)
        new_obj.skills = other.skills.copy()  # shallow copy of list
        return new_obj

emp1 = Employee("Aju", 10000)
emp1.skills.append("Python")

emp2 = Employee.from_employee(emp1)
print(emp2.name)    # Aju
print(emp2.skills)  # ['Python']
```

**B) Using Python's `copy` module**

```python
emp3 = copy.copy(emp1)      # shallow copy — nested objects are shared
emp4 = copy.deepcopy(emp1)  # deep copy — fully independent clone
```

> **Shallow copy** copies the object but shares nested mutable objects. **Deep copy** creates fully independent copies of everything.

---

## Constructor Overloading

Python allows only one `__init__`. Overloading is simulated using three techniques:

```python
class Employee:
    # Technique 1: Default parameter values
    # Technique 2: *args (positional) and **kwargs (keyword)
    def __init__(self, name="Unknown", salary=0, *args, **kwargs):
        self.name = name
        self.salary = salary
        self.skills = list(args)                   # extra positional args → list
        self.department = kwargs.get("department") # optional named arg

    # Technique 3: Class methods as alternative constructors
    @classmethod
    def from_string(cls, data):
        name, salary = data.split(",")
        return cls(name.strip(), int(salary.strip()))

    @classmethod
    def intern(cls, name):
        return cls(name, 0)  # interns start at salary 0

    def display(self):
        print(f"Name: {self.name}, Salary: {self.salary}, "
              f"Skills: {self.skills}, Dept: {self.department}")

# Different ways to create the same class
emp1 = Employee()                                          # all defaults
emp2 = Employee("Aju", 10000)                             # positional args
emp3 = Employee("Hari", 12000, "Python", "SQL", department="IT")  # *args + **kwargs
emp4 = Employee.from_string("Aju, 15000")                 # from string
emp5 = Employee.intern("Hari")                            # intern shortcut

emp3.display()
# Name: Hari, Salary: 12000, Skills: ['Python', 'SQL'], Dept: IT
```

**Advantages:**
- Flexibility to initialize objects in multiple ways
- No code duplication — single `__init__` handles all cases
- Cleaner code compared to defining multiple constructors

---

## Constructor Chaining
<img width="1440" height="466" alt="image" src="https://github.com/user-attachments/assets/9e4cb758-a7c2-4ec7-bb80-f87a6953f93f" />

Constructor chaining means the child class calls the parent class constructor using `super().__init__()` to reuse initialization logic instead of repeating it.**Without chaining (bad practice — code duplication):**

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, roll_no):
        self.name = name    # repeated from Person
        self.age = age      # repeated from Person
        self.roll_no = roll_no

s = Student("Aju", 20, 101)
s.display()
```

**With chaining (correct approach):**

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class Student(Person):
    def __init__(self, name, age, roll_no):
        super().__init__(name, age)  # delegates to Person
        self.roll_no = roll_no

    def display(self):
        print(self.name, self.age, self.roll_no)

s = Student("Hari", 20, 101)
s.display()   # Hari 20 101
```

**Key points:**
- `super().__init__()` must be called explicitly in Python — it is not automatic
- Best practice: call `super().__init__()` at the top of the child constructor
- In multiple inheritance, Python uses **MRO (Method Resolution Order)** to determine which parent's constructor gets called

> **Interview tip:** If you inherit from a class but don't call `super().__init__()`, the parent's attributes will not be initialized on the child object, leading to `AttributeError` when accessed.
