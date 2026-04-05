## Object-Oriented Programming (OOP)

OOP is a way of writing code by organizing it around **objects** — things that hold both data and actions together. Think of it like designing a real-world blueprint before building the actual thing.

**Two core building blocks:**

- **Class** — the blueprint/template (e.g., "Car")
- **Object** — the actual thing built from that blueprint (e.g., your specific red Honda)

---

## OOP vs Procedural Programming

Procedural programming is the older style — you write steps one after another, like a recipe. OOP thinks in terms of real-world things instead of just steps.
<img width="1440" height="720" alt="image" src="https://github.com/user-attachments/assets/82644fe4-1e8e-4a97-97aa-9c092493b366" />


## The 4 Pillars of OOP

These are the concepts interviewers love to ask about. Learn them with the real-world angle.
<img width="1440" height="848" alt="image" src="https://github.com/user-attachments/assets/bbce439b-07a7-49b7-86dc-533d4e756d0a" />


## Why OOP? (Interview-Ready Answer)

OOP shines for large, complex systems because of four key strengths:

**Modularity** — Break a big system into smaller, independent parts (classes). Each class handles one responsibility. Easy to find, fix, and update things. Example: a bank app has separate classes for `Account`, `Customer`, `Transaction`.

**Code Reusability** — Write once, extend many times using inheritance. `Car` and `Bike` both inherit from `Vehicle` — you don't rewrite the common engine logic twice.

**Scalability** — Adding a new feature means adding a new class, not rewriting old ones. The system grows without breaking existing parts.

**Security** — Sensitive data (like a bank balance) is kept private inside objects. Only controlled methods like `deposit()` or `withdraw()` can touch it.

---

## Real-World Analogy — A Bank

| OOP Concept | Bank Example |
|---|---|
| Class | Blueprint for a bank Account |
| Object | Raj's specific account |
| Attribute | Account number, balance, name |
| Method | `deposit()`, `withdraw()`, `transfer()` |
| Encapsulation | Balance is private — only `withdraw()` can change it |
| Inheritance | `SavingsAccount` extends `Account` |
| Polymorphism | `transfer()` works differently for domestic vs international |
| Abstraction | You use an ATM without knowing the internal processing |

---

## Quick Recall Cheat Sheet

- **Class** = blueprint &nbsp;|&nbsp; **Object** = instance of that blueprint
- **4 Pillars** = Encapsulation · Inheritance · Polymorphism · Abstraction
- **OOP vs Procedural** = real-world modeling vs step-by-step instructions
- **Why OOP?** = Modularity · Reusability · Scalability · Security
