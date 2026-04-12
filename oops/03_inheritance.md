
<img width="1440" height="1102" alt="image" src="https://github.com/user-attachments/assets/10755947-6a90-4e03-8087-5bccfa619985" />

# Python Inheritance 

## What is Inheritance?

Inheritance lets a **child class** acquire the attributes and methods of a **parent class** — avoiding code duplication and modelling real-world "is-a" relationships.

```
School → Student  ("Student is-a School member")
Animal → Dog      ("Dog is-a Animal")
```

---

## Core Vocabulary

| Term | Meaning |
|---|---|
| Parent / Super / Base class | The class being inherited from |
| Child / Sub / Derived class | The class that inherits |
| `super()` | Built-in proxy to access parent class methods/attributes |

---

## 1. Single Inheritance

One child inherits from one parent.

```python
class Company:
    def __init__(self, name):
        self.__name = name          # private attribute

    def get_company_name(self):
        return self.__name          # public getter for private attr

class Employee(Company):
    def __init__(self, company_name, emp_name, emp_id):
        super().__init__(company_name)   # call parent __init__
        self.emp_name = emp_name
        self.emp_id   = emp_id

    def get_details(self):
        return f"{self.emp_name} | {self.emp_id} | {self.get_company_name()}"

e = Employee("Google", "Aju", "E001")
print(e.get_company_name())   # Google
print(e.get_details())        # Aju | E001 | Google
```

---

## 2. Multiple Inheritance

One child inherits from two or more parents. Python resolves method lookup order via **MRO** (Method Resolution Order — left to right, depth first).

```python
class Parent1:
    def method1(self): return "From Parent1"

class Parent2:
    def method2(self): return "From Parent2"

class Child(Parent1, Parent2):
    pass

c = Child()
print(c.method1())  # From Parent1
print(c.method2())  # From Parent2
print(Child.__mro__)  # Shows resolution order
```

> **Diamond Problem:** If `B` and `C` both inherit from `A` and override a method, and `D` inherits from `B, C` — which version does `D` get? Python uses MRO to resolve this deterministically (left-to-right, C3 linearisation). Java avoids this entirely by disallowing multiple class inheritance.

---

## 3. Multilevel Inheritance

A chain: Grandparent → Parent → Child.

```python
class Grandparent:
    def grandparent_method(self): return "Grandparent"

class Parent(Grandparent):
    def parent_method(self): return "Parent"

class Child(Parent):
    def child_method(self): return "Child"

c = Child()
print(c.grandparent_method())  # Grandparent  ← accessible!
print(c.parent_method())       # Parent
print(c.child_method())        # Child
```

---

## 4. Hierarchical Inheritance

Multiple children share one parent.

```python
class School:
    def info(self): return "Part of this school"

class Student(School):
    def study(self): return "Aju is studying"

class Teacher(School):
    def teach(self): return "Hari is teaching"

s = Student()
t = Teacher()
print(s.info())   # Part of this school  ← shared from School
print(t.info())   # Part of this school  ← same shared method
```

---

## 5. Hybrid Inheritance

A mix of inheritance types — here, multilevel + multiple combined.

```python
class GrandParent:
    def gp_method(self): return "GrandParent"

class Parent1(GrandParent):    # multilevel part
    def p1_method(self): return "Parent1"

class Parent2:
    def p2_method(self): return "Parent2"

class Child(Parent1, Parent2):  # multiple part
    pass

c = Child()
print(c.gp_method())   # GrandParent  (accessed via Parent1 chain)
print(c.p1_method())   # Parent1
print(c.p2_method())   # Parent2
```

---

## Key Concept: `super()` and Attribute Overriding

<img width="1440" height="656" alt="image" src="https://github.com/user-attachments/assets/7560e2ef-96af-4e3d-ae51-b7337ee9e2be" />


```python
class Company:
    class_which_is = "parent class attr"
    def __init__(self, name):
        self.__name = name
        self.which_is = "parent instance attr"

    def get_company_name(self):
        return self.__name

class Employee(Company):
    class_which_is = "child class attr"    # overrides parent's class attr

    def __init__(self, company_name, emp_name, emp_id):
        super().__init__(company_name)
        self.emp_name = emp_name
        self.emp_id   = emp_id
        self.which_is = "child instance attr"  # overrides parent's instance attr

    def demo(self):
        print(self.class_which_is)         # "child class attr"
        print(super().class_which_is)      # "parent class attr"  ✅

        # print(super().which_is)          # AttributeError! ❌
        # super() doesn't expose instance attributes

        print(self.get_company_name())     # "Google" ✅ via public method
        # print(self._Employee__name)      # works but breaks encapsulation

e = Employee("Google", "Aju", "E001")
e.demo()
```

---

## Method Overriding

A child class provides its own version of a method already defined in the parent. The child version runs instead of the parent's when called on a child object.

**Conditions that must match (all 5):**
same method name · same parameters · same return type · same or less-restrictive access · method must be inheritable (not private)

```python
class Animal:
    def sound(self):
        return "Some generic sound"

class Dog(Animal):
    def sound(self):          # same name, same params → overrides
        return "Dog barks"

class Cat(Animal):
    def sound(self):
        return "Cat meows"

dog, cat = Dog(), Cat()
print(dog.sound())                       # Dog barks
print(cat.sound())                       # Cat meows
print(super(Dog, dog).sound())           # Some generic sound  ← parent's version
```

> **Python 3.12+:** You can use `@override` from `typing` for clarity, though Python doesn't enforce it at runtime.

---

## Method Overriding vs Method Overloading

| | Overriding | Overloading |
|---|---|---|
| Where | Between parent and child | Within the same class |
| Needs inheritance? | Yes | No |
| Parameters | Must match parent | Must differ |
| Resolved at | Runtime (dynamic) | Compile time (static) |
| Python support | Full support | No native support (use `*args` / default args) |

```python
# Python "overloading" pattern — use default args
class Calculator:
    def add(self, a, b, c=0):
        return a + b + c

calc = Calculator()
print(calc.add(2, 3))     # 5
print(calc.add(2, 3, 4))  # 9
```

---

## Access Modifiers in Inheritance

```python
class Parent:
    def __init__(self):
        self.public    = "anyone"       # accessible everywhere
        self._protected = "convention"  # convention: internal use (not enforced)
        self.__private  = "mangled"     # name-mangled → _Parent__private

class Child(Parent):
    def access_demo(self):
        print(self.public)          # ✅
        print(self._protected)      # ✅ (by convention only)
        # print(self.__private)     # ❌ AttributeError
        # print(super().__private)  # ❌ AttributeError

c = Child()
print(c._Parent__private)          # ✅ name mangling workaround (avoid in production)
```

---

## Interview Questions

**Q1: Can a subclass access private members of its parent?**
No — not directly. Private attributes are name-mangled (e.g. `__name` → `_ClassName__name`). Access them through a public method the parent provides.

**Q2: What is `super()` used for?**
Two purposes: (1) call the parent's `__init__` to set up inherited state, (2) access parent class methods/class attributes that have been overridden. It does not work with instance attributes.

**Q3: What is the Diamond Problem?**
When `D` inherits from `B` and `C`, both of which inherit from `A`, there's ambiguity about which `A`'s method version `D` uses. Python resolves it using **C3 MRO** (left-to-right). Java avoids it by disallowing multiple class inheritance (only interfaces).

**Q4: What are the 5 conditions for method overriding?**
Same name, same parameters, same return type, same number of parameters, and same parameter types. If any differ — it's overloading, not overriding.

**Q5: `self` vs `super()` when accessing class attributes?**
`self.attr` → checks child first, then parent (child overrides parent). `super().attr` → goes directly to parent's class attribute. For instance attributes, `super()` gives an `AttributeError` — it only proxies class-level members.

---

## Why Use Inheritance?

**Reusability** — write common logic once in the parent; all children reuse it.
**Modularity** — each class handles a focused responsibility.
**Extensibility** — add new behaviour in a child without touching the parent.
**Maintainability** — fix a bug once in the parent; all children benefit automatically.
