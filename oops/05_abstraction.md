<img width="1440" height="660" alt="image" src="https://github.com/user-attachments/assets/e56b8db2-4121-4522-9bae-97b6992388a1" />

# Python Abstraction, Static, Class & Default Methods 

## What is Abstraction?

Abstraction means **showing what an object does, hiding how it does it**. You expose a clean interface to the user and keep the implementation details internal.

Real-world analogy: you press the accelerator in a car — you don't need to know how fuel injection, combustion, and the drivetrain work. The *what* (go faster) is exposed; the *how* is hidden.

In OOP, abstraction is enforced via **abstract classes** — a blueprint that says "every subclass MUST implement these methods" without defining the implementation itself.

---

## Without Abstraction — the Problem

```python
class Shape:
    def area(self):
        pass              # just returns None — no enforcement

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    # forgot to implement area() — no error!

c = Circle(5)
print(c.area())   # None  ← silent bug, no warning at all
```

Python lets you create the object and call the method — it just silently returns `None`. This is dangerous in large codebases.

---

## Abstraction with `abc` Module

Python achieves strict abstraction using the `abc` (Abstract Base Class) module. Import `ABC` and `abstractmethod`.

```python
from abc import ABC, abstractmethod

class Shape(ABC):                   # inherit from ABC
    @abstractmethod
    def area(self):                 # abstract method — no body
        pass

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):                 # MUST implement this
        return 3.14 * self.radius ** 2

c = Circle(5)
print(c.area())   # 78.5  ✅
```

Now if a subclass forgets to implement `area()`:

```python
class Square(Shape):
    def __init__(self, side):
        self.side = side
    # area() NOT implemented

s = Square(4)
# TypeError: Can't instantiate abstract class Square
# without an implementation for abstract method 'area'
```

Python throws an error the moment you try to create the object — not when you call the method. This catches bugs early.

---

## Abstract Class with Concrete Methods

An abstract class can have both abstract methods (no body) and concrete methods (with body). Subclasses inherit the concrete methods for free.

```python
from abc import ABC, abstractmethod

class Shape(ABC):

    def greeting(self):              # concrete method — has a body
        print("I am a shape.")       # inherited by all subclasses

    @abstractmethod
    def area(self):                  # abstract — must be overridden
        pass

class Rectangle(Shape):
    def __init__(self, l, w):
        self.l = l
        self.w = w

    def area(self):
        return self.l * self.w

r = Rectangle(4, 5)
r.greeting()       # I am a shape.  ← from abstract class
print(r.area())    # 20             ← from Rectangle
```

---

## Can you instantiate an abstract class directly?

```python
s = Shape()
# TypeError: Can't instantiate abstract class Shape
# without an implementation for abstract method 'area'
```

No — abstract classes are blueprints only. But they can have a constructor (`__init__`) that subclass constructors call via `super()`.

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, name):
        self.name = name            # constructor — sets shared state

    @abstractmethod
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return f"{self.name} says: Bark!"

d = Dog("Aju")
print(d.speak())    # Aju says: Bark!
```

---

## Can an abstract class extend another abstract class?

Yes. The child abstract class doesn't have to implement the parent's abstract methods — it can leave them for its own concrete subclasses.

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    @abstractmethod
    def fuel_type(self): pass

class ElectricVehicle(Vehicle, ABC):   # still abstract, adds more
    @abstractmethod
    def battery_capacity(self): pass

class Tesla(ElectricVehicle):
    def fuel_type(self):         return "Electric"
    def battery_capacity(self):  return "100 kWh"

t = Tesla()
print(t.fuel_type())          # Electric
print(t.battery_capacity())   # 100 kWh
```

---

## Python vs Java — Abstraction

| Feature | Java | Python (`abc`) |
|---|---|---|
| Keyword | `abstract` (built-in) | `ABC` + `@abstractmethod` |
| Enforced at | Compile time | Runtime |
| Strictness | Always strict | Optional (can skip `abc`) |
| Instantiation block | Compile error | `TypeError` at runtime |
| Interfaces | Native (`interface`) | No direct equivalent — use `ABC` |

---

## Method Types in Python

Now let's look at the three types of methods in a Python class.

<img width="1440" height="600" alt="image" src="https://github.com/user-attachments/assets/1551d491-bf41-4f7f-8dbc-f588d5cc072e" />


## Instance Method (default)

The most common method type. Takes `self` as the first argument — gives access to both instance attributes and class attributes.

```python
class Dog:
    species = "Canine"            # class variable

    def __init__(self, name):
        self.name = name          # instance variable

    def bark(self):               # instance method
        return f"{self.name} barks! Species: {self.species}"

d = Dog("Aju")
print(d.bark())    # Aju barks! Species: Canine
```

---

## `@classmethod`

Takes `cls` (the class itself) as the first argument instead of `self`. Can read and modify class-level state. Cannot access instance attributes.

The most common use case is **factory methods** — alternative constructors that create instances in different ways.

```python
class DatabaseConfig:
    connection_limit = 100        # class variable, shared by all instances

    def __init__(self, db_name):
        self.db_name = db_name

    @classmethod
    def update_limit(cls, new_limit):     # modify class state
        cls.connection_limit = new_limit

    @classmethod
    def from_url(cls, url):               # factory method — alternative constructor
        db_name = url.split("/")[-1]
        return cls(db_name)

DatabaseConfig.update_limit(200)
print(DatabaseConfig.connection_limit)    # 200 — changed for ALL instances

db = DatabaseConfig.from_url("postgres://localhost/myapp")
print(db.db_name)    # myapp
```

---

## `@staticmethod`

No `self` or `cls`. Just a regular function that lives inside the class's namespace for organisational clarity. Cannot access instance or class state unless explicitly passed.

Use it for **utility/helper functions** that logically belong to the class but don't need any class or instance data.

```python
class DatabaseConfig:
    @staticmethod
    def get_version():
        return "v2.0.0"

    @staticmethod
    def validate_port(port):
        return 1024 <= port <= 65535

print(DatabaseConfig.get_version())        # v2.0.0
print(DatabaseConfig.validate_port(5432))  # True
print(DatabaseConfig.validate_port(80))    # False
```

---

## All three together — a complete example

```python
from abc import ABC, abstractmethod

class Employee(ABC):
    company = "Acme Corp"         # class variable

    def __init__(self, name, salary):
        self.name   = name
        self.salary = salary

    def get_details(self):        # instance method
        return f"{self.name} | {self.salary} | {self.company}"

    @classmethod
    def change_company(cls, new_name):    # class method
        cls.company = new_name

    @staticmethod
    def is_valid_salary(salary):          # static method
        return salary > 0

    @abstractmethod
    def role(self):                       # abstract method
        pass

class Developer(Employee):
    def role(self):
        return "Software Developer"

class Manager(Employee):
    def role(self):
        return "Engineering Manager"

d = Developer("Aju",  80000)
m = Manager("Hari", 120000)

print(d.get_details())            # Aju | 80000 | Acme Corp
print(m.role())                   # Engineering Manager

Employee.change_company("TechCo") # changes for ALL instances
print(d.get_details())            # Aju | 80000 | TechCo  ← updated!

print(Employee.is_valid_salary(50000))   # True
print(Employee.is_valid_salary(-100))    # False
```

---

## "Default Methods" (Python vs Java)

Java 8 introduced `default` methods in interfaces to add new methods without breaking existing implementations. Python has no direct equivalent concept.

In Python, **regular methods in a base class serve the same purpose** — subclasses inherit them automatically without needing to override.

```python
class Printable:
    def display(self):                    # acts like a Java default method
        print(f"Object: {self.__class__.__name__}")

class Report(Printable):
    pass                                  # inherits display() for free

class Invoice(Printable):
    def display(self):                    # override if you need custom behaviour
        print(f"Invoice from Aju Corp")

r = Report()
i = Invoice()

r.display()    # Object: Report   ← inherited
i.display()    # Invoice from Aju Corp  ← overridden
```

---

## `@staticmethod` vs `@classmethod` — When to Use Which

```
Need to access/modify instance data?   → instance method (self)
Need to access/modify class variables? → @classmethod (cls)
Need neither, but logically part of the class? → @staticmethod
```

```python
class MathUtils:

    pi = 3.14159                    # class variable

    @staticmethod
    def square(n):                  # pure utility, no class/instance needed
        return n * n

    @classmethod
    def circle_area(cls, r):        # uses class variable pi
        return cls.pi * r * r

print(MathUtils.square(4))          # 16
print(MathUtils.circle_area(5))     # 78.53975
```

---

## Member Lifecycle — Static vs Instance

| | Class variable / static method | Instance variable / method |
|---|---|---|
| Ownership | Class level | Object level |
| Lifecycle | Exists for the whole program | Exists as long as the object |
| Access | `ClassName.attr` | `object.attr` |
| Shared | Yes — all instances see same value | No — each object has its own copy |

```python
class Counter:
    count = 0               # class variable — shared

    def __init__(self):
        Counter.count += 1
        self.id = Counter.count    # instance variable — unique per object

a = Counter()
b = Counter()
c = Counter()

print(Counter.count)   # 3  — shared across all
print(a.id, b.id, c.id)  # 1 2 3  — unique per object
```

---

## Interview Questions

**Q1: What is abstraction and how is it different from encapsulation?**
Abstraction hides *what is unnecessary* — you expose only the interface. Encapsulation hides *internal data* using access modifiers. Abstraction is about design; encapsulation is about data protection.

**Q2: How do you enforce abstraction in Python?**
Inherit from `ABC` and decorate methods with `@abstractmethod`. Any subclass that doesn't implement all abstract methods will raise a `TypeError` when you try to instantiate it.

**Q3: Can an abstract class have a constructor?**
Yes. The constructor runs when a subclass object is created, via `super().__init__(...)`. You cannot directly instantiate the abstract class itself.

**Q4: What is the difference between `@classmethod` and `@staticmethod`?**
`@classmethod` receives `cls` — the class itself — and can read/modify class-level state. `@staticmethod` receives nothing implicitly and cannot access class or instance state. Use `@classmethod` for factory methods or when modifying class variables; use `@staticmethod` for pure utility functions grouped under the class.

**Q5: Why would you use a `@staticmethod` instead of a standalone function?**
For logical organisation — when a helper function is closely related to the class's purpose but doesn't need any class or instance data, putting it as a static method keeps the namespace clean and makes the code more readable.

**Q6: What is a factory method? Give an example.**
A class method used as an alternative constructor — creates an instance from a different input format. Example: `Employee.from_csv("Aju,80000")` parses the string and returns an `Employee` object, keeping the creation logic inside the class.
