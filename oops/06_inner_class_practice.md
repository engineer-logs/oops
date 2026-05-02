# Inner Classes in Python

---

## 🧱 Basics You Should Know

1. **Class Definition**: A class is a blueprint for creating objects with attributes (data) and methods (behavior).
2. **Encapsulation**: Hiding internal details using private attributes (`__name`) to protect data.
3. **Properties**: Python decorators (`@property`, `@setter`) that let you control how attributes are read/written.
4. **Type Hints**: Annotations like `name: str` that document expected types (Python 3.5+).
5. **Static Methods**: Methods that don't use `self`; they belong to the class, not instances.
6. **Exceptions**: Custom errors like `ValueError` signal when input is structurally correct but semantically invalid.

---

## 📑 Table of Contents

1. [What is an Inner Class?](#what-is-an-inner-class)
2. [Why Use Inner Classes?](#why-use-inner-classes)
3. [Inner Class Fundamentals](#inner-class-fundamentals)
4. [The Company-Employee Practice Example](#the-company-employee-practice-example)
5. [Key Standards & Best Practices](#key-standards--best-practices)
6. [Interview Questions & Answers](#interview-questions--answers)
7. [Advanced Topics & Pointers](#advanced-topics--pointers)

---

## What is an Inner Class?

An **inner class** (also called a **nested class**) is a class defined inside another class. It logically groups classes that belong together, increases encapsulation, and keeps namespaces clean by avoiding name clashes.

**In simple terms**: You define a class inside another class when that inner class is only used by the outer class.

### Key Benefits

- **Encapsulation**: Inner class is part of the outer class's interface.
- **Logical Grouping**: Related functionality stays together.
- **Namespace Management**: Avoids polluting the global namespace.
- **Access**: Inner class can access private members of the outer class.

### Basic Syntax

```python
class Outer:
    class Inner:
        def __init__(self):
            self.value = 42

# Usage
inner_obj = Outer.Inner()
print(inner_obj.value)  # Output: 42
```

---

## Why Use Inner Classes?

### Problem: Without Inner Classes

```python
# Without inner classes, you'd have:
class Employee:
    def __init__(self, id, name, company):
        self.id = id
        self.name = name
        self.company = company

class Company:
    def __init__(self, name):
        self.name = name
        self.employees = []

# Now: Employee and Company are separate, even though Employee exists only for Company
```

### Solution: With Inner Classes

```python
class Company:
    class Employee:
        def __init__(self, id, name):
            self.id = id
            self.name = name
    
    def __init__(self, name):
        self.name = name
        self.employees = []

# Now: Employee is logically part of Company
```

**Why this is better:**
- ✅ Clearly shows Employee belongs to Company
- ✅ Prevents someone from accidentally using Employee independently
- ✅ Keeps the namespace clean

---

## Inner Class Fundamentals

### 1. Accessing Outer Class from Inner Class

An inner class can access the outer class's attributes and methods, but it needs a reference to the outer instance.

```python
class Company:
    def __init__(self, name):
        self.__name = name
    
    def get_company_info(self):
        return f"Company: {self.__name}"
    
    class Employee:
        def __init__(self, company_ref, name):
            self.__company = company_ref  # Reference to outer instance
            self.__name = name
        
        def show_company(self):
            # Access outer class's private method
            return self.__company.get_company_info()

# Usage
company = Company("TechCorp")
emp = company.Employee(company, "Aju")
print(emp.show_company())  # Output: Company: TechCorp
```

**Key Point**: The inner class must store a reference (`company: Company`) to access the outer class.

---

### 2. Type Hints with Forward References

When an inner class needs to reference its outer class in type hints, use `from __future__ import annotations`.

```python
from __future__ import annotations

class Company:
    class Employee:
        def __init__(self, company: Company, name: str) -> None:
            self.__company = company
            self.__name = name
```

**Why `from __future__ import annotations`?**
- Makes type hints lazy (stored as strings, not evaluated immediately).
- Allows `Company` reference before the class is fully defined.
- Available in Python 3.7+.

**Alternative (Python 3.10+):**
```python
from typing import Optional
# Instead of: Optional[str]
# You can use: str | None
```

---

### 3. Private Methods & Validation

Use `__` prefix for truly private methods. Static methods are ideal for validation since they don't need instance data.

```python
class Employee:
    @staticmethod
    def __validate_name(value: str, field: str) -> str:
        """Private static method for validation."""
        if not isinstance(value, str) or not value.strip():
            raise ValueError(f"{field} must be non-empty string")
        return value.strip()
    
    def __init__(self, name: str):
        self.__name = self.__validate_name(name, "name")
```

**Key Points:**
- `@staticmethod` doesn't access `self`; it's a utility function.
- Double underscore (`__`) makes methods truly private (name-mangled).
- Raise `ValueError` for invalid values (correct type, wrong content).

---

## The Company-Employee Practice Example

Now let's break down the complete, production-grade implementation:

### Overview of the Structure

```
Company (outer class)
├── __name, __location, __employees
├── properties: name, location
├── CRUD methods: add_employee, get_employee, edit_employee, remove_employee
├── utilities: promote_employee, find_employees_by_name
├── reporting: list_employees, employee_count, total_payroll
│
└── Employee (inner class)
    ├── __id, __name, __position, __email, __salary, __active
    ├── properties: id, name, position, email, salary, active
    ├── methods: deactivate, get_company_info, get_employee_info, to_dict
```

---

### Part 1: Imports & Forward Reference Setup

```python
from __future__ import annotations
from typing import Optional

# from __future__ import annotations:
#   Allows forward references (Company) before it's fully defined.
#   Makes all type hints lazy (strings).
#   Python 3.7+ feature.

# from typing import Optional:
#   Optional[T] means T or None.
#   In Python 3.10+, you can write: T | None instead.
```

**Interview Tip**: Understand the difference between `Optional[str]` and `Optional[str] = None`:
- `Optional[str]` → value CAN be None, but parameter is still required.
- `Optional[str] = None` → value CAN be None, and parameter is optional.

```python
# ❌ Optional doesn't make parameter optional:
def greet(name: Optional[str]):
    print(name)
greet()  # Error: missing argument

# ✅ Default value makes it optional:
def greet(name: Optional[str] = None):
    print(name)
greet()  # Works: prints None
```

---

### Part 2: The Employee Inner Class

```python
class Company:
    class Employee:
        def __init__(
            self,
            company: Company,
            employee_id: int,
            name: str,
            position: str,
            email: str,
            salary: float,
        ) -> None:
            self.__company: Company = company
            self.__id: int = employee_id
            self.__name: str = self.__require_non_empty(name, "name")
            self.__position: str = self.__require_non_empty(position, "position")
            self.__email: str = self.__require_non_empty(email, "email")
            self.__salary: float = self.__require_non_negative(salary, "salary")
            self.__active: bool = True
```

**What's happening:**
1. `company: Company` → Outer class reference (to access company methods later).
2. All attributes are private (`__`), protected by validation.
3. Validation happens at assignment time, not later.

---

### Part 3: Static Validation Methods

```python
@staticmethod
def __require_non_empty(value: str, field_name: str) -> str:
    '''
    Validates that a string is non-empty and not just whitespace.
    Raises ValueError if validation fails.
    '''
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{field_name} must be a non-empty string")
    return value.strip()

@staticmethod
def __require_non_negative(value: float, field_name: str) -> float:
    if value < 0:
        raise ValueError(f"{field_name} must be >= 0")
    return float(value)
```

**Breaking it down:**

1. **`isinstance(value, str)`**: Type checking built-in. Returns `True` if `value` is a string.
   - Why use `isinstance`? It respects inheritance (unlike `type()`).
   - `isinstance(5, int)` → `True`
   - `isinstance(5, (int, float))` → `True` (checks multiple types)

2. **`value.strip()`**: Removes leading/trailing whitespace.
   - `"  hello  ".strip()` → `"hello"`
   - Handles empty strings and whitespace-only strings.

3. **`ValueError`**: Raised when type is correct but value is invalid.
   - `int("abc")` → `ValueError` (string format is correct, but content isn't a number)
   - Your code: type is `str`, but value is empty → `ValueError`

**Interview Insight**: Why static methods for validation?
- ✅ No need for `self`; they're utility functions.
- ✅ Can be called on the class itself: `Employee.__require_non_empty(...)`
- ✅ Pure functions (no side effects).

---

### Part 4: Properties (Getters & Setters)

```python
@property
def name(self) -> str:
    return self.__name

@name.setter
def name(self, value: str) -> None:
    self.__name = self.__require_non_empty(value, "name")
```

**How it works:**

1. **`@property`** → Creates a **getter**. Lets you read the attribute like a normal attribute.
   ```python
   emp.name  # Calls the getter, returns self.__name
   ```

2. **`@name.setter`** → Adds a **setter** to the same property. Lets you assign with validation.
   ```python
   emp.name = "Hari"  # Calls the setter, validates, then assigns
   ```

**Why this pattern?**
- ✅ **Encapsulation**: Data is protected (`__name` is private).
- ✅ **Validation**: Every assignment is validated.
- ✅ **Clean API**: Users write `emp.name` not `emp.get_name()` or `emp.set_name()`.

**Read-only property** (no setter):
```python
@property
def id(self) -> int:
    return self.__id

# emp.id = 5  # ❌ Error: can't set attribute
```

**Computed property** (derived from other data):
```python
@property
def is_senior(self) -> bool:
    return self.__salary > 150000  # Derived, not stored
```

---

### Part 5: Inner Class Methods

```python
def deactivate(self) -> None:
    """Soft delete: mark as inactive without removing."""
    self.__active = False

def get_company_info(self) -> str:
    """Access outer class method via stored reference."""
    return self.__company.get_company_info()

def get_employee_info(self) -> str:
    """Formatted summary of employee data."""
    status = "Active" if self.__active else "Inactive"
    return (
        f"[{self.__id}] {self.__name} | {self.__position} | "
        f"{self.__email} | ${self.__salary:,.2f} | {status}"
    )

def to_dict(self) -> dict[str, object]:
    """Convert to dictionary (useful for JSON serialization)."""
    return {
        "id": self.__id,
        "name": self.__name,
        "position": self.__position,
        "email": self.__email,
        "salary": self.__salary,
        "active": self.__active,
    }
```

**Key patterns:**
- `deactivate()` → **Soft delete** (marks inactive instead of removing).
- `get_company_info()` → Shows inner class accessing outer class.
- `to_dict()` → Serialization pattern (prepare for JSON/API).

**Modern type hint** `dict[str, object]`:
- Python 3.9+: Use `dict[K, V]` instead of `Dict[K, V]` from `typing`.
- `object` is the base type (accepts any type of value).

---

### Part 6: Outer Class (Company)

```python
class Company:
    def __init__(self, name: str, location: str) -> None:
        self.__name: str = self.__require_non_empty(name, "name")
        self.__location: str = self.__require_non_empty(location, "location")
        self.__employees: dict[int, Company.Employee] = {}
        self.__next_employee_id: int = 1
```

**Type annotation breakdown:**
- `dict[int, Company.Employee]` → Dictionary where:
  - **Key**: Employee ID (integer)
  - **Value**: Employee object (referenced as `Company.Employee`)

---

### Part 7: CRUD Operations

#### CREATE: Adding Employees

```python
def add_employee(
    self, name: str, position: str, email: str, salary: float
) -> Employee:
    employee = self.Employee(
        company=self,
        employee_id=self.__next_employee_id,
        name=name,
        position=position,
        email=email,
        salary=salary,
    )
    self.__employees[self.__next_employee_id] = employee
    self.__next_employee_id += 1
    return employee
```

**Key points:**
- `company=self` → Passes outer instance to inner class.
- Auto-incrementing ID: `__next_employee_id` ensures unique IDs.
- Return type is `Employee` (shorthand for `Company.Employee` inside the class).

#### READ: Getting an Employee

```python
def get_employee(self, employee_id: int) -> Optional[Employee]:
    return self.__employees.get(employee_id)
```

**Why `Optional[Employee]`?**
- Employee might not exist → return `None`.
- Caller must check before using:
  ```python
  emp = company.get_employee(1)
  if emp is not None:
      print(emp.name)
  ```

#### UPDATE: Editing an Employee

```python
def edit_employee(
    self,
    employee_id: int,
    *,
    name: Optional[str] = None,
    position: Optional[str] = None,
    email: Optional[str] = None,
    salary: Optional[float] = None,
) -> bool:
    employee = self.get_employee(employee_id)
    if employee is None:
        return False

    if name is not None:
        employee.name = name
    if position is not None:
        employee.position = position
    if email is not None:
        employee.email = email
    if salary is not None:
        employee.salary = salary
    return True
```

**Important syntax: Keyword-only arguments**

The `*` in `def edit_employee(..., *, name=None, ...):` means:
- All parameters after `*` MUST be passed by name (not positionally).
- ❌ `company.edit_employee(1, "Aju", "Manager")` → Error
- ✅ `company.edit_employee(1, name="Aju", position="Manager")` → Works

**Why?** Prevents confusion with positional args. Makes the API clearer.

#### DELETE: Removing Employees

```python
def remove_employee(self, employee_id: int, soft_delete: bool = True) -> bool:
    employee = self.get_employee(employee_id)
    if employee is None:
        return False

    if soft_delete:
        employee.deactivate()  # Mark inactive, keep record
    else:
        del self.__employees[employee_id]  # Actually remove from dict
    return True
```

**Two deletion strategies:**
- **Soft delete** (default): Keep record, mark inactive. Better for audit trails.
- **Hard delete**: Actually remove from storage. Data is gone.

---

### Part 8: Advanced Operations

```python
def promote_employee(
    self, employee_id: int, new_position: str, salary_increment: float = 0.0
) -> bool:
    employee = self.get_employee(employee_id)
    if employee is None:
        return False
    employee.position = new_position
    if salary_increment:
        employee.salary = employee.salary + salary_increment
    return True

def find_employees_by_name(self, query: str) -> list[Employee]:
    q = query.strip().lower()
    return [e for e in self.__employees.values() if q in e.name.lower()]
```

**Patterns:**
- **Promotion**: Combines position change + salary increment (optional).
- **Search**: Case-insensitive substring matching.

---

### Part 9: Reporting & Analytics

```python
def list_employees(self, active_only: bool = True) -> list[str]:
    employees = self.__employees.values()
    if active_only:
        employees = [e for e in employees if e.active]
    return [e.get_employee_info() for e in employees]

def employee_count(self, active_only: bool = True) -> int:
    if not active_only:
        return len(self.__employees)
    return sum(1 for e in self.__employees.values() if e.active)

def total_payroll(self, active_only: bool = True) -> float:
    employees = self.__employees.values()
    if active_only:
        employees = [e for e in employees if e.active]
    return sum(e.salary for e in employees)
```

**Patterns:**
- **`list_employees()`** → String formatted output (for display).
- **`employee_count()`** → Aggregate count using `sum(1 for ...)`.
- **`total_payroll()`** → Aggregate sum using `sum(e.salary for ...)`.

All support filtering by `active_only` (respects soft-deleted employees).

---

## Key Standards & Best Practices

### 1. Type Hints (Modern Python Style)

```python
# ❌ Old style (Python 3.5-3.8)
from typing import Dict, List, Optional
def func(data: Dict[int, str]) -> Optional[List[str]]:
    pass

# ✅ Modern style (Python 3.9+)
def func(data: dict[int, str]) -> list[str] | None:
    pass
```

**Key changes:**
- `dict` instead of `Dict`
- `list` instead of `List`
- `X | None` instead of `Optional[X]`

### 2. Private Attributes Convention

```python
# ✅ Single underscore: "private by convention"
self._internal_value = 42  # Developers: don't use this

# ✅ Double underscore: "truly private" (name-mangled)
self.__secret_value = 42  # Python renames to _ClassName__secret_value
```

**Use double underscore** for:
- Attributes you want to prevent accidental access to.
- Class internals that should never be touched.

### 3. Properties for Controlled Access

```python
class BankAccount:
    def __init__(self, balance: float):
        self.__balance = balance
    
    @property
    def balance(self) -> float:
        """Read-only: check balance."""
        return self.__balance
    
    @property
    def can_withdraw(self) -> bool:
        """Computed property: derived from state."""
        return self.__balance > 0
    
    def withdraw(self, amount: float) -> bool:
        """Method: modify state with business logic."""
        if amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
```

**Rule of thumb:**
- **Properties** → For simple getters/setters with validation.
- **Methods** → For complex operations (withdrawal, transfer, etc.).

### 4. Error Handling Best Practices

```python
# ❌ Too broad
try:
    employee.name = user_input
except Exception:
    print("Something failed")

# ✅ Specific to expected error
try:
    employee.name = user_input
except ValueError as e:
    print(f"Invalid input: {e}")
```

### 5. Method Return Values

Design your API to signal success/failure:

```python
# Pattern 1: Boolean return
if company.edit_employee(1, name="Aju"):
    print("Edit successful")
else:
    print("Employee not found")

# Pattern 2: Return object on success, None on failure
emp = company.get_employee(1)
if emp:
    print(emp.name)
```

---

## Interview Questions & Answers

### Q1: Why are there two `__require_non_empty` methods (one in Employee, one in Company)?

**Answer:**

Each class maintains its own validation logic for **encapsulation** and **low coupling**.

**Why not reuse one?**

If Employee called `Company.__require_non_empty()`:
- ❌ Employee would depend on Company.
- ❌ Tight coupling: changes to Company validation affect Employee.
- ❌ Violates single responsibility principle.

**Current design advantage:**
- ✅ Each class is self-contained.
- ✅ You can change Employee validation separately from Company validation.
- ✅ No cross-class dependencies.

**When should you reuse?**

If validation is truly shared business logic:

```python
# Option 1: Utility module
def validate_non_empty(value: str, field: str) -> str:
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{field} must be non-empty")
    return value.strip()

class Company:
    def __init__(self, name: str):
        self.__name = validate_non_empty(name, "name")

class Employee:
    def __init__(self, name: str):
        self.__name = validate_non_empty(name, "name")
```

**Interview-level takeaway**: Same logic, different responsibility → duplication is intentional and acceptable design.

---

### Q2: What's the purpose of storing a reference to `Company` in `Employee.__init__`?

**Answer:**

The inner class needs access to the outer instance to:

1. **Call outer methods**: `self.__company.get_company_info()`
2. **Access outer data**: `self.__company.__name`
3. **Maintain relationship**: Employee knows which company it belongs to.

```python
class Employee:
    def __init__(self, company: Company, ...):
        self.__company = company  # Store reference
    
    def get_company_info(self):
        return self.__company.get_company_info()  # Use reference
```

**Without the reference:**
- Employee can't access Company's methods/attributes.
- Inner classes don't automatically get outer instance.

---

### Q3: Why use `soft_delete` instead of always hard-deleting?

**Answer:**

Soft delete (marking inactive) is better for:

1. **Audit trails**: Keep history of all employees ever employed.
2. **Data integrity**: Foreign keys from other tables still work.
3. **Recovery**: Mistakes can be undone.
4. **Reporting**: See "Employees ever employed" vs "Current employees".

```python
# Soft delete
company.remove_employee(1, soft_delete=True)
company.list_employees(active_only=False)  # Still see inactive employees

# Hard delete
company.remove_employee(1, soft_delete=False)
company.list_employees(active_only=False)  # Employee is gone
```

---

### Q4: What does `*` do in function parameters?

**Answer:**

The `*` forces all parameters after it to be **keyword-only**:

```python
def edit_employee(self, employee_id: int, *, name=None, position=None):
    pass

# ❌ Positional: Error
edit_employee(1, "Aju", "Manager")

# ✅ Keyword: Works
edit_employee(1, name="Aju", position="Manager")
```

**Why?** Makes the API clearer and prevents argument order mistakes.

---

### Q5: What's the difference between `@property` and `@staticmethod`?

| Feature | `@property` | `@staticmethod` |
|---------|-----------|-----------------|
| Accesses `self`? | Yes | No |
| Called with `()`? | No—`obj.name` | Yes—`Class.method()` |
| Used for? | Getters/setters | Utility functions |
| Can be overridden? | Yes | No |
| Example | `emp.name` | `Employee.__validate()` |

---

### Q6: Why use `isinstance()` instead of `type()`?

**Answer:**

`isinstance()` respects inheritance; `type()` doesn't.

```python
class Animal: pass
class Dog(Animal): pass

d = Dog()

type(d) == Animal      # False ❌
isinstance(d, Animal)  # True ✔

# isinstance is correct when checking parent classes
```

**Use `isinstance()` for type checking** (it's safer and more Pythonic).

---

### Q7: What's `from __future__ import annotations`?

**Answer:**

Makes type hints lazy (stored as strings) instead of evaluated immediately. This allows forward references.

```python
# Without: NameError (Company not defined yet)
class Company:
    class Employee:
        def __init__(self, company: Company):  # ❌ Error
            pass

# With from __future__ import annotations:
from __future__ import annotations

class Company:
    class Employee:
        def __init__(self, company: Company):  # ✅ Works!
            pass
```

**When to use:** Always, when you have forward references or circular dependencies.

---
![](https://github.com/engineer-logs/oops/blob/main/assets/employee_company.svg)

---

## Advanced Topics & Pointers

1. **Multiple Inner Classes** — A single outer class can have multiple inner classes for different responsibilities (e.g., `Company.Employee`, `Company.Department`, `Company.Payroll`).

2. **Static Inner Class Pattern** — Use `@staticmethod` inside inner classes for utility functions that don't need outer instance access.

3. **Abstract Inner Classes** — Combine `ABC` (Abstract Base Class) with inner classes to enforce contracts for child classes.

4. **Metaclasses & Inner Classes** — Use `type()` to dynamically create inner classes at runtime (advanced metaprogramming).

5. **Serialization & Deserialization** — Extend `to_dict()` pattern with `from_dict()` classmethod for loading objects from JSON/databases.

6. **Context Managers for Inner Classes** — Implement `__enter__()` and `__exit__()` to use inner classes with `with` statements for resource management.

---

## Quick Reference: Common Patterns

### Pattern 1: Validation at Assignment

```python
@property
def salary(self) -> float:
    return self.__salary

@salary.setter
def salary(self, value: float) -> None:
    self.__salary = self.__require_non_negative(value, "salary")
```

### Pattern 2: Optional Parameters with Defaults

```python
def edit_employee(
    self,
    employee_id: int,
    *,
    name: Optional[str] = None,
    salary: Optional[float] = None,
) -> bool:
    emp = self.get_employee(employee_id)
    if emp is None:
        return False
    if name is not None:
        emp.name = name
    if salary is not None:
        emp.salary = salary
    return True
```

### Pattern 3: Soft Delete with Active Flag

```python
def remove_employee(self, employee_id: int, soft_delete: bool = True) -> bool:
    emp = self.get_employee(employee_id)
    if emp is None:
        return False
    if soft_delete:
        emp.deactivate()
    else:
        del self.__employees[employee_id]
    return True
```

### Pattern 4: Filtering Collections

```python
def list_employees(self, active_only: bool = True) -> list[str]:
    employees = self.__employees.values()
    if active_only:
        employees = [e for e in employees if e.active]
    return [e.get_employee_info() for e in employees]
```

### Pattern 5: Serialization

```python
def to_dict(self) -> dict[str, object]:
    return {
        "id": self.__id,
        "name": self.__name,
        "position": self.__position,
        "salary": self.__salary,
        "active": self.__active,
    }
```

---

## Example Usage: Full Workflow

```python
# Create company
company = Company("Tech Innovators", "Silicon Valley")

# Add employees
e1 = company.add_employee("Aju Singh", "Software Engineer", "aju@tech.com", 120000)
e2 = company.add_employee("Hari Patel", "Product Manager", "hari@tech.com", 135000)

# Edit (property setters with validation)
company.edit_employee(e1.id, position="Senior Software Engineer", salary=140000)

# Promote with salary increment
company.promote_employee(e2.id, "Senior Product Manager", salary_increment=10000)

# Display info
print(company.get_company_info())
# Output: Company: Tech Innovators | Location: Silicon Valley | Active Employees: 2

print("Employees:", company.list_employees())
# Output: Employees: [
#   [1] Aju Singh | Senior Software Engineer | aju@tech.com | $140,000.00 | Active
#   [2] Hari Patel | Senior Product Manager | hari@tech.com | $145,000.00 | Active
# ]

print(f"Total Payroll: ${company.total_payroll():,.2f}")
# Output: Total Payroll: $285,000.00

# Soft delete (mark inactive, keep record)
company.remove_employee(e1.id, soft_delete=True)
print(company.list_employees(active_only=False))
# Shows Aju as Inactive, Hari as Active

# Search by name
results = company.find_employees_by_name("Hari")
print(f"Found: {[e.name for e in results]}")
# Output: Found: ['Hari Patel']
```

---

## Summary: What You've Learned

✅ **Basics**: What inner classes are and why they exist
✅ **Design**: How to reference outer class from inner class
✅ **Types**: Modern type hints with forward references
✅ **Properties**: Getters and setters for controlled access
✅ **Validation**: Static methods for input validation
✅ **CRUD**: Creating, reading, updating, deleting data
✅ **Patterns**: Soft delete, filtering, serialization
✅ **Best Practices**: Encapsulation, low coupling, clear APIs
✅ **Interview Prep**: Answers to common questions

---

## 🚀 Interview Confidence Checklist

Before your interview, verify you can explain:

- [ ] What an inner class is and why you'd use one
- [ ] How to pass outer instance reference to inner class
- [ ] Difference between `@property` and `@staticmethod`
- [ ] Why `isinstance()` is better than `type()`
- [ ] When to use `ValueError` vs other exceptions
- [ ] Purpose of `from __future__ import annotations`
- [ ] Difference between soft delete and hard delete
- [ ] What `*` does in function parameters (keyword-only)
- [ ] Why duplicate validation methods is acceptable design
- [ ] How to serialize objects to dictionaries
