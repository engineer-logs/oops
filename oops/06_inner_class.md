# Inner Classes (Nested Classes) in Python

---

## 🧱 Basics You Should Know

* **Classes** — blueprints for objects; can be defined inside other classes or methods
* **Static members** — class-level variables/methods accessed via `ClassName.member`, not tied to instances
* **Instance members** — object-level attributes/methods that exist per instance
* **Scope** — determines where a variable or class is accessible; nested classes have limited scope
* **Reference passing** — Python doesn't auto-bind outer instances; explicit passing is required
* **Inheritance** — `@classmethod` uses `cls` to support subclass creation; `@staticmethod` hardcodes the class

---

## Table of Contents

1. [Static Nested Classes](#static-nested-classes)
2. [Non-Static Inner Classes](#non-static-inner-classes)
3. [Local Inner Classes](#local-inner-classes)
4. [Anonymous Classes & Dynamic Class Creation](#anonymous-classes--dynamic-class-creation)
5. [Key Differences: Python vs Java](#key-differences-python-vs-java)

---

## Static Nested Classes

### Concept

A **static nested class** is defined inside another class but does not require an instance of the outer class. It can access only the outer class's static members (class variables), not instance variables. Unlike Java, Python nested classes don't automatically access outer static members—you must reference them explicitly (e.g., `Outer.salary`).

**💡 Interview Tip:** When deciding between **direct instantiation** vs **@classmethod factory**, ask: "Do I need conditional logic?" If no, keep it simple with direct instantiation. If yes, use `@classmethod` for extensibility and inheritance support.

### Code Example

```python
class Outer:
    salary = 800000
    bonus_percent = 15
    
    class Inner:
        """Static nested class accessing outer class variables."""
        
        def __init__(self, title="Standard", include_bonus=False):
            self.title = title
            self.include_bonus = include_bonus
        
        def display(self):
            """Display salary information."""
            base_salary = Outer.salary  # Explicit reference required
            
            if self.include_bonus:
                bonus = base_salary * (Outer.bonus_percent / 100)
                total = base_salary + bonus
                print(f"[{self.title}] Base: ${base_salary:,} + Bonus (15%): ${bonus:,.0f} = ${total:,.0f}")
            else:
                print(f"[{self.title}] Salary: ${base_salary:,}")

# APPROACH 1: Direct Instantiation (Simple Cases)
# Use when no initialization logic is needed
obj = Outer.Inner(title="Developer")
obj.display()  # [Developer] Salary: $800,000

# APPROACH 2: @classmethod Factory (Conditional Logic)
# Use when creation logic varies based on parameters

@classmethod
def create_inner(cls, level="standard", with_bonus=False):
    """Factory method with conditional logic."""
    level_config = {
        "junior": ("Junior Developer", False),
        "senior": ("Senior Developer", False),
        "manager": ("Manager", True),
        "executive": ("Executive", True),
    }
    
    title, include_bonus = level_config.get(level, ("Standard", with_bonus))
    return cls.Inner(title=title, include_bonus=include_bonus)

# Usage
junior = Outer.create_inner("junior")
junior.display()  # [Junior Developer] Salary: $800,000

manager = Outer.create_inner("manager")
manager.display()  # [Manager] Base: $800,000 + Bonus (15%): $120,000 = $920,000
```

### Why @classmethod Over @staticmethod?

```python
# ❌ @staticmethod — Hardcodes outer class, breaks inheritance
class Outer:
    @staticmethod
    def create():
        return Outer.Inner()  # Always creates Outer.Inner, not subclass's Inner

# ✅ @classmethod — Uses cls, supports inheritance
class Outer:
    @classmethod
    def create(cls):
        return cls.Inner()  # Creates proper subclass's Inner

class SubOuter(Outer):
    class Inner:
        pass

obj = SubOuter.create()  # Correctly creates SubOuter.Inner
```
![nested](https://github.com/engineer-logs/oops/blob/main/assets/static_nested.svg)
---

## Non-Static Inner Classes

### Concept

A **non-static inner class** (nested class with instance binding) is associated with an instance of the outer class. In Java, non-static inner classes implicitly access the outer instance. In Python, there is no implicit reference—you must explicitly pass the outer instance to access its instance variables and methods.

**💡 Interview Tip:** In Python, you have two patterns: (1) pass outer instance to methods, or (2) store it in the constructor. Pattern 2 is cleaner if you need frequent access.

### Code Example

```python
# Pattern 1: Pass outer instance to individual methods
class Outer:
    def __init__(self, company_name: str):
        self.company_name = company_name
    
    class Inner:
        def __init__(self, employee_name: str):
            self.employee_name = employee_name
        
        def display_info(self, outer_instance):
            # Access outer instance variables explicitly
            print(f"Company: {outer_instance.company_name}, Employee: {self.employee_name}")

outer = Outer("TechCorp")
inner = outer.Inner("Aju Kumar")
inner.display_info(outer)  # Company: TechCorp, Employee: Aju Kumar

# Pattern 2: Store outer instance in constructor (cleaner)
class Outer1:
    def __init__(self, company_name: str):
        self.company_name = company_name
    
    class Inner:
        def __init__(self, employee_name: str, outer_instance: 'Outer1'):
            self.employee_name = employee_name
            self.outer_instance = outer_instance  # Store reference
        
        def display_info(self):
            # Access via stored reference
            print(f"Company: {self.outer_instance.company_name}, Employee: {self.employee_name}")

outer1 = Outer1("DataSoft")
inner1 = outer1.Inner("Hari Nair", outer1)
inner1.display_info()  # Company: DataSoft, Employee: Hari Nair
```
![](https://github.com/engineer-logs/oops/blob/main/assets/non_static.svg)
---

## Local Inner Classes

### Concept

A **local inner class** is defined inside a method of the outer class. It has access to the method's local variables and the outer class's static members, but not instance variables (unless passed explicitly). Local inner classes are scoped to that method only—instances cannot be created outside it. Unlike Java, Python allows modifying local variables in the enclosing method.

**💡 Interview Tip:** Local inner classes are useful for encapsulating method-specific logic, but keep them simple. If complexity grows, extract to a module-level class.

### Code Example

```python
class Outer:
    class_var = "Outer Class Variable"
    
    def __init__(self, company_name: str):
        self.company_name = company_name
    
    def method_with_inner_class(self, value):
        """Method containing a local inner class."""
        x = 10  # Local variable of the method
        
        class LocalInner:
            """Local inner class — scoped to this method."""
            
            def process(self):
                # Access method's local variables
                print(f"Processing value: {value}")
                print(f"Local method variable: {x}")
                # Access outer class static member
                print(f"Class variable: {Outer.class_var}")
        
        x += 1  # Modify local variable (Python allows this; Java requires 'final')
        inner = LocalInner()
        inner.process()

# Usage
outer = Outer("TechCorp")
outer.method_with_inner_class(42)
# Output:
# Processing value: 42
# Local method variable: 11
# Class variable: Outer Class Variable

# ❌ This fails — LocalInner is not accessible outside the method
# obj = outer.LocalInner()  # NameError: name 'LocalInner' is not defined
```

![](https://github.com/engineer-logs/oops/blob/main/assets/local3.svg)
---

## Anonymous Classes & Dynamic Class Creation

### Concept

Python doesn't have a direct *anonymous class* construct like Java, but you can simulate it using:

1. **`type()` function** — Dynamic class creation at runtime (closest to Java's anonymous classes)
2. **Inline usage** — Create and use a class in one expression
3. **`types.SimpleNamespace`** — Lightweight object for structured data (often cleaner)
4. **Lambdas and functions** — Python's preferred alternative (simpler and more Pythonic)

In practice, Python developers avoid anonymous classes and prefer functions or simple closures.

**💡 Interview Tip:** Mention that Python's philosophy is different from Java: "functions are first-class objects," so lambdas and closures solve most cases where Java uses anonymous classes.

### Code Example

```python
# Method 1: type() — Dynamic class creation (metaprogramming)
Person = type("Person", (), {
    "name": "Aju",
    "greet": lambda self: f"Hello, {self.name}"
})

p = Person()
print(p.greet())  # Hello, Aju

# Method 2: Truly inline (one-liner)
obj = type("Temp", (), {"x": 10})()
print(obj.x)  # 10

# Method 3: SimpleNamespace (cleaner for simple data)
from types import SimpleNamespace

person = SimpleNamespace(name="Hari", age=25, city="Kerala")
print(person.name)  # Hari
print(person.age)   # 25

# Method 4: Lambda + closure (Python-preferred for behavior)
greet = lambda name: f"Hello, {name}!"
print(greet("Aju"))  # Hello, Aju!

# Method 5: Local class (not truly anonymous, but scoped)
def create_person():
    class Person:
        def __init__(self, name):
            self.name = name
        
        def greet(self):
            return f"Hi, I'm {self.name}"
    
    return Person("Hari")

person = create_person()
print(person.greet())  # Hi, I'm Hari
```

### When to Use Each Approach

```python
# ✅ Use type() for metaprogramming, frameworks (Django serializers, ORMs)
# ❌ Avoid for regular application code — hard to read and maintain

# ✅ Use SimpleNamespace for lightweight data containers
# ❌ Don't use for complex behavior

# ✅ Use lambda/closure for small callbacks, filters, decorators
# ❌ Don't use for multi-line logic

# ✅ Use local classes for scoped, method-specific logic
# ❌ Don't use for widely shared types
```
![](https://github.com/engineer-logs/oops/blob/main/assets/anonymous.svg)
---

## Key Differences: Python vs Java

### Quick Comparison Table

| Feature | Java | Python |
|---------|------|--------|
| **Static nested class** | Auto-accesses outer static members | Must use explicit reference (e.g., `Outer.salary`) |
| **Non-static inner class** | Implicit outer instance reference | Must pass outer instance explicitly |
| **Factory pattern** | Not needed for nesting | Use `@classmethod` for inheritance support |
| **@staticmethod vs @classmethod** | N/A | `@classmethod` is preferred (uses `cls`) |
| **Local inner class** | Can access `final` variables | Can access & modify local variables freely |
| **Anonymous class** | Common idiom | Rare; use lambda/closure/SimpleNamespace |
| **Scope visibility** | Strict access control (private/protected) | Everything is accessible; scope-based convention |

### Code Comparison

```python
# ===== JAVA =====
// Static nested class — auto-accesses outer static
class Outer {
    static int salary = 800000;
    
    static class Inner {
        void display() {
            System.out.println(salary);  // Direct access
        }
    }
}

// Non-static inner class — implicit outer reference
class Outer {
    String name = "Corp";
    
    class Inner {
        void show() {
            System.out.println(name);  // Implicit access via 'Outer.this.name'
        }
    }
}

// ===== PYTHON =====
# Static nested class — explicit reference needed
class Outer:
    salary = 800000
    
    class Inner:
        def display(self):
            print(Outer.salary)  # Explicit reference required

# Non-static inner class — explicit parameter
class Outer:
    def __init__(self, name):
        self.name = name
    
    class Inner:
        def __init__(self, outer):
            self.outer = outer
        
        def show(self):
            print(self.outer.name)  # Explicit reference
```

---

## 🚀 Advanced Topic Hints

* **Metaclasses** — Use `type()` to create classes dynamically; enables framework features like Django models
* **Descriptors & Properties** — Advanced attribute access control; combines with nested classes in framework design
* **Context Managers (`__enter__`, `__exit__`)** — Pair with inner classes for resource management patterns
* **Multiple Inheritance & MRO** — Complex inheritance with nested classes; watch for method resolution order conflicts
* **Dataclasses & Named Tuples** — Modern alternatives to SimpleNamespace for structured data
* **Protocol Classes (typing)** — Structural subtyping for inner classes in type hints

---

## 📝 Key Takeaways for Interviews

✅ **Direct instantiation** for simple nested classes (no logic needed)

✅ **@classmethod factory** for conditional creation & inheritance support

✅ **Always pass outer instance explicitly** in Python (unlike Java's implicit binding)

✅ **Local inner classes** for method-scoped logic; access to method variables but not instance variables (without explicit passing)

✅ **Avoid anonymous classes** — use lambda, SimpleNamespace, or local classes instead

✅ **Python's philosophy:** Simplicity over formality; functions are first-class objects, so leverage them

---

## 💻 Quick Reference Code

```python
# Static nested class with @classmethod factory
class Outer:
    salary = 100000
    
    class Inner:
        def __init__(self, title, with_bonus=False):
            self.title = title
            self.with_bonus = with_bonus
        
        def calc(self):
            return Outer.salary * 1.15 if self.with_bonus else Outer.salary
    
    @classmethod
    def create(cls, level):
        levels = {"junior": ("Dev", False), "manager": ("Mgr", True)}
        title, bonus = levels.get(level, ("Standard", False))
        return cls.Inner(title, bonus)

# Non-static with stored reference
class Company:
    def __init__(self, name):
        self.name = name
    
    class Employee:
        def __init__(self, emp_name, company):
            self.emp_name = emp_name
            self.company = company
        
        def display(self):
            print(f"{self.emp_name} @ {self.company.name}")

# Local inner class
class Processor:
    def process(self, data):
        counter = 0
        
        class DataHandler:
            def handle(self):
                nonlocal counter
                counter += 1
                return f"Processing {data}, count: {counter}"
        
        handler = DataHandler()
        return handler.handle()
```

---
