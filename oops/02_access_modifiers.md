## What are Access Modifiers?

Access modifiers define **how visible and accessible** a class member (attribute or method) is to code outside that class. They are the mechanism through which encapsulation is enforced — by controlling who can read or modify what.

> **Python vs other languages:** Python does not have strict access modifier keywords like Java's `public`, `private`, `protected`. Instead, it uses **naming conventions** that signal intent. The interpreter enforces only light restrictions (name mangling for `__`), not hard blocks.

---

## Python Module vs Package

Before diving in, a quick clarification used in scope discussions:

```
📁 my_project/         ← package (a folder with __init__.py)
 ├── 📄 library.py     ← module (a single .py file)
 ├── 📄 book.py        ← module
```

- A **module** = one `.py` file
- A **package** = a folder containing multiple modules

"Accessible within the same module" means within the same `.py` file. Python has no true package-level access restriction like Java does.

---

## The Three Access Levels in Python

<img width="1440" height="424" alt="image" src="https://github.com/user-attachments/assets/9f6eb5d0-3409-4c5b-a142-0178ed2d461c" />


## 1. Public Access

No underscore prefix. Accessible from **anywhere** — inside the class, subclasses, other modules, or any external code.

**Scenario:** A `Student` class where name and roll number are public — anyone can read or set them directly.

```python
class Student:
    def __init__(self, name, roll_no):
        self.name = roll_no        # public attribute
        self.roll_no = roll_no     # public attribute

    def display(self):             # public method
        print("Name:", self.name, "| Roll No:", self.roll_no)

s = Student("Aju", 101)
s.name = "Hari"           # accessible globally — direct modification
s.display()               # accessible globally
```

**Key points:**
- Default in Python — if you don't add any underscore, the member is public
- Best for data that is genuinely meant to be used by external code
- Used in APIs and interfaces

---

## 2. Private Access

Double underscore prefix `__`. Python applies **name mangling** — `self.__seats` becomes `self._Bus__seats` internally. This makes accidental access from outside harder, but not impossible.

**Scenario:** A `Bus` class where seat count is internal — only the class controls how it changes.

```python
class Bus:
    def __init__(self, route, total_seats):
        self.route = route                  # public
        self.__total_seats = total_seats    # private — internal only
        self.__booked = 0                   # private

    def book_seat(self):
        if self.__booked < self.__total_seats:
            self.__booked += 1
            print(f"Seat booked on route {self.route}.")
        else:
            print("No seats available.")

    def get_availability(self):             # public getter — controlled access
        return self.__total_seats - self.__booked

b = Bus("Route 12", 40)
b.book_seat()
print("Available seats:", b.get_availability())

# b.__total_seats   →  AttributeError (name mangling hides it)
# But technically accessible as: b._Bus__total_seats (not recommended)
```

**Key points:**
- Signals "this is internal — do not touch from outside"
- Encourages getter/setter methods for controlled access
- Not inherited — subclasses cannot access `__` attributes directly
- Python's name mangling is a soft restriction, not a hard lock

---

## 3. Protected Access

Single underscore prefix `_`. This is a **convention only** — Python does not enforce any restriction. It signals: "intended for internal use or subclasses, not for general outside code."

**Scenario:** A `Vehicle` base class with a `_fuel_type` attribute meant to be shared with subclasses like `Truck`.

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand           # public
        self._fuel_type = None       # protected — for subclass use

    def _display_fuel(self):         # protected method
        print(f"{self.brand} runs on {self._fuel_type}")

class Truck(Vehicle):
    def __init__(self, brand, payload):
        super().__init__(brand)
        self._fuel_type = "Diesel"   # accessible in subclass
        self.payload = payload

    def info(self):
        self._display_fuel()         # using protected method from parent
        print(f"Payload: {self.payload} tons")

t = Truck("Aju Logistics", 10)
t.info()
# Truck runs on Diesel
# Payload: 10 tons
```

**Key points:**
- Single `_` is a **hint to developers**, not enforced by Python
- Used when a parent class wants to share internals with subclasses but not with all external code
- Still technically accessible from outside — `t._fuel_type` works — but it's bad practice

---

## 4. Default (No modifier) — Python context

In Java, "default" means package-private. In Python, there is no built-in package-level restriction. Any attribute with no underscore is effectively public across the entire program.

**Scenario:** A `Classroom` utility with a plain method — accessible from anywhere within the project.

```python
class Classroom:
    def announce(self, message):    # default = public in Python
        print("Announcement:", message)

c = Classroom()
c.announce("Exam on Monday")       # accessible from any module
```

> **Interview note:** When asked about "default" access in Python, clarify that Python has no package-private concept. All no-underscore members are public. The `_` and `__` conventions are Python's way of approximating protected and private.

---

## Comparison Table

<img width="1440" height="618" alt="image" src="https://github.com/user-attachments/assets/04b82320-0846-4942-ae05-40a248042da5" />


## Quick Reference — Naming Conventions

| Syntax | Level | Enforcement |
|---|---|---|
| `self.title` | Public | None — open to all |
| `self._genre` | Protected | Convention only — "for internal/subclass use" |
| `self.__price` | Private | Name-mangled to `_ClassName__price` |

---

## Key Takeaways for Interviews

- Python uses **naming conventions**, not keywords, for access control
- `__name` triggers **name mangling** — `self.__seats` becomes `self._Bus__seats` — making outside access non-obvious but not impossible
- `_name` is a gentleman's agreement: "don't use this externally" — Python won't stop you
- Python has **no true package-level access** (no equivalent to Java's default/package-private)
- Private members are **not visible to subclasses** — a child class cannot access `self.__attr` defined in the parent directly
- Access modifiers are about **communicating design intent** as much as enforcing restriction
