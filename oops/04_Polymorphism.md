# Python Polymorphism 
<img width="1440" height="680" alt="image" src="https://github.com/user-attachments/assets/433a1d1b-fa9a-4ac3-b2b9-93a4d8296f9a" />

## What is Polymorphism?

**One interface, many behaviours.** The word comes from Greek — *poly* (many) + *morphe* (form). In OOP, the same method name behaves differently depending on the object that calls it.

```
animal.speak()  →  Dog: "barks"
animal.speak()  →  Cat: "meows"
animal.speak()  →  Animal: "generic sound"
```

Same method name. Different behaviour. That's polymorphism.

---

## Type 1 — Compile-time Polymorphism (Method Overloading)

Resolved at compile time. Multiple methods share the same name but have different parameters.

**Python does NOT support this natively.** When you define two methods with the same name, the second one silently overwrites the first.

```python
class Calculator:
    def add(self, a, b):
        return a + b

    def add(self, a, b, c):    # overwrites the first add!
        return a + b + c

calc = Calculator()
# calc.add(2, 3)   → TypeError: missing argument 'c'
print(calc.add(2, 3, 4))      # 9 — only this works now
```

### Python's workarounds for overloading

**Option 1 — Default arguments** (recommended for simple cases)

```python
class Calculator:
    def add(self, a, b, c=0):   # c defaults to 0
        return a + b + c

calc = Calculator()
print(calc.add(2, 3))        # 5  (c=0 used)
print(calc.add(2, 3, 4))     # 9  (c=4 used)
```

**Option 2 — `*args`** (recommended when count of inputs is unknown)

```python
class Calculator:
    def add(self, *args):        # accepts any number of arguments
        return sum(args)

calc = Calculator()
print(calc.add(2, 3))            # 5
print(calc.add(2, 3, 4))         # 9
print(calc.add(1, 2, 3, 4, 5))   # 15
```

**Option 3 — `**kwargs`** (when you also need named arguments)

```python
class Profile:
    def display(self, name, **kwargs):
        info = f"Name: {name}"
        for key, val in kwargs.items():
            info += f", {key}: {val}"
        return info

p = Profile()
print(p.display("Aju"))                           # Name: Aju
print(p.display("Hari", age=25, city="Kollam"))   # Name: Hari, age: 25, city: Kollam
```

---

## Type 2 — Runtime Polymorphism (Method Overriding)

Resolved at runtime. A child class provides its own version of a method already defined in the parent. Python determines which version to run based on the actual object type — not the reference type.

```python
class Animal:
    def speak(self):
        return "Animal speaks"

class Dog(Animal):
    def speak(self):        # overrides parent
        return "Dog barks"

class Cat(Animal):
    def speak(self):        # overrides parent
        return "Cat meows"

dog = Dog()
cat = Cat()

print(dog.speak())                    # Dog barks
print(cat.speak())                    # Cat meows
print(super(Dog, dog).speak())        # Animal speaks  ← parent's version
```

### The real power — polymorphic behaviour via a common interface

This is what makes polymorphism truly useful. You can write code that works on any object as long as it has the right method — without caring about the specific type.

```python
animals = [Dog(), Cat(), Animal()]

for animal in animals:
    print(animal.speak())

# Dog barks
# Cat meows
# Animal speaks
```

One loop. Three different behaviours. This is runtime polymorphism in action.

---

## Polymorphism with Functions (Duck Typing)

Python extends polymorphism beyond classes through **duck typing** — if an object has the method you need, it works, regardless of its class.

> "If it walks like a duck and quacks like a duck, it's a duck."

```python
class Dog:
    def speak(self): return "Bark"

class Cat:
    def speak(self): return "Meow"

class Robot:                  # not related to Animal at all!
    def speak(self): return "Beep boop"

def make_it_speak(entity):    # works on anything with .speak()
    print(entity.speak())

make_it_speak(Dog())     # Bark
make_it_speak(Cat())     # Meow
make_it_speak(Robot())   # Beep boop
```

---

## Polymorphism with Built-in Operators (Operator Overloading)

Python's operators are polymorphic by design. The `+` operator behaves differently depending on the types involved. You can extend this to your own classes using **dunder methods** (double-underscore methods).

```python
print(2 + 3)          # 5        (int addition)
print("Hi" + " Aju")  # Hi Aju   (string concatenation)
print([1,2] + [3,4])  # [1,2,3,4] (list join)

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):        # overloads the + operator
        return Point(self.x + other.x, self.y + other.y)

    def __str__(self):               # overloads str() / print()
        return f"Point({self.x}, {self.y})"

p1 = Point(1, 2)
p2 = Point(3, 4)
print(p1 + p2)     # Point(4, 6)
```

Common dunder methods for operator overloading:

| Operator | Dunder method |
|---|---|
| `+` | `__add__` |
| `-` | `__sub__` |
| `*` | `__mul__` |
| `==` | `__eq__` |
| `<` | `__lt__` |
| `str()` / `print()` | `__str__` |
| `len()` | `__len__` |

---

## Polymorphism with `len()` — built-in example

```python
print(len("Aju"))         # 3  — counts characters
print(len([1, 2, 3, 4]))  # 4  — counts elements
print(len({"a": 1}))      # 1  — counts keys

class Team:
    def __init__(self, members):
        self.members = members

    def __len__(self):
        return len(self.members)

team = Team(["Aju", "Hari", "Arjun"])
print(len(team))    # 3
```

---

## Key Comparison — Overloading vs Overriding---

## Interview Questions

**Q1: What is polymorphism? Give a real-world analogy.**
A single action behaving differently based on context. Real-world analogy: the `+` key on a calculator adds numbers, but the same key concept on a keyboard inputs a character. Same interface, different behaviour based on what it's applied to.

**Q2: Does Python support method overloading?**
Not natively. When two methods share a name, the second overwrites the first. Simulate overloading using default arguments (`c=0`) or variable-length arguments (`*args`).

**Q3: What is the difference between overloading and overriding?**
Overloading = same class, same name, different parameters, resolved at compile time (not native in Python). Overriding = child class redefines a parent's method, same name and parameters, resolved at runtime (fully supported in Python).

**Q4: What is duck typing?**
Python's approach to polymorphism — if an object has the required method, Python will call it regardless of the object's class. No explicit interface or inheritance required.

**Q5: What are dunder methods? Give an example.**
Double-underscore methods like `__add__`, `__str__`, `__len__` let you define how built-in operators and functions behave with your custom objects. Example: defining `__add__` lets your class use the `+` operator.

**Q6: How does runtime polymorphism work in Python?**
Python resolves the method to call based on the actual object type at runtime. A variable holding a `Dog` instance will call `Dog.speak()`, not `Animal.speak()`, even if the variable was typed as `Animal`. This is enabled by Python's dynamic dispatch mechanism.

---

## Quick Mental Model

```
Polymorphism
├── Same method name
├── Different behaviour depending on object/args
│
├── Overloading (compile-time)
│   └── Python: use *args / default args
│
├── Overriding (runtime)
│   └── Python: child class redefines parent method
│
├── Duck Typing
│   └── No class relationship needed — just the right method
│
└── Operator Overloading
    └── Dunder methods: __add__, __str__, __len__, etc.
```
