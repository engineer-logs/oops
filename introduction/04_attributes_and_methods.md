
## Attributes

Attributes are the **data an object holds** — they represent the state of the object at any point in time. Think of them as the object's memory.

For a `BankAccount` class, two attributes make sense:

- `name` — stores the account holder's name (string)
- `balance` — stores the money in the account (number)

Since `balance` is personal information, it should be **private** — hidden from the outside world. No one should be able to directly read or change another person's balance.

```python
class BankAccount:
    def __init__(self, name, balance):
        self.__name = name        # private — hidden from outside
        self.__balance = balance  # private — sensitive data
```

The `__` (double underscore) prefix makes the attribute private in Python. Trying to access `obj.__balance` directly from outside the class will throw an error.

---

## Methods

Methods are **functions defined inside a class** — they define what the object can *do*. They usually read or modify the object's attributes in a controlled way.

For a `BankAccount`, the useful actions are: check balance, deposit money, and withdraw money.

```python
class BankAccount:
    def __init__(self, name, balance):
        self.__name = name
        self.__balance = balance

    # Getter — read the name
    def getName(self):
        return self.__name

    # Setter — update the name
    def setName(self, name):
        self.__name = name

    # Getter — read the balance
    def getBalance(self):
        return self.__balance

    # Deposit money
    def deposit(self, amount):
        self.__balance += amount

    # Withdraw money (with validation)
    def withdrawal(self, amount):
        if amount > self.__balance:
            print("Insufficient balance")
            return False
        self.__balance -= amount
        return True
```

Notice how `__balance` is never touched directly from outside — all access goes through methods. This is the whole point of encapsulation.

---

## Getters and Setters

These are the two most important method types in real-world OOP projects:

- **Getter** — reads a private attribute and returns it. Example: `getBalance()`
- **Setter** — updates a private attribute after validation. Example: `setName()`

They exist because private attributes cannot be accessed directly — getters and setters are the controlled "doors" into that data.

```python
acc = BankAccount("Aju", 5000)

# Getter — reading private data through a method
print(acc.getName())      # Output: Aju
print(acc.getBalance())   # Output: 5000

# Setter — updating private data through a method
acc.setName("Hari")
print(acc.getName())      # Output: Hari

# Direct access — this will FAIL
print(acc.__balance)      # AttributeError!
```
<img width="1440" height="678" alt="image" src="https://github.com/user-attachments/assets/952399a1-d85a-40c6-be16-61630b856ce4" />


The outside world cannot touch private attributes directly. All interaction must go through the public methods — which act as controlled gates.

---

## How Attributes and Methods Interact

Let's trace through a full usage example to see everything working together:

```python
# Create account for Aju with balance 5000
acc1 = BankAccount("Aju", 5000)

# Deposit 2000 — balance becomes 7000
acc1.deposit(2000)
print(acc1.getBalance())    # Output: 7000

# Withdraw 3000 — balance becomes 4000
acc1.withdrawal(3000)
print(acc1.getBalance())    # Output: 4000

# Try to overdraw — blocked by validation inside the method
acc1.withdrawal(9000)       # Output: Insufficient balance

# Final balance
print(acc1.getBalance())    # Output: 4000
```

The `withdrawal()` method validates the amount before touching `__balance`. This is why private attributes + methods is a safer design — the method acts as a guard.

Here's the flow of a withdrawal call:
<img width="1440" height="698" alt="image" src="https://github.com/user-attachments/assets/a190bfb9-34d3-4812-9b53-76dcbca7a35f" />

## Important Points to Remember

**Accessing attributes** — always use getters and setters for private attributes. Never rely on direct access.

**Encapsulation** — keep attributes private, expose only what is needed through methods. In Python, `__attributeName` signals it is private by convention (name mangling enforces it somewhat, but Python trusts the programmer).

**Default values** — in Python, all attributes must be initialized inside `__init__`. There is no concept of default fields like some other languages. If you skip initialization, the attribute simply won't exist on the object.

**Validating inputs** — always check inside methods before modifying data:

```python
def deposit(self, amount):
    if amount <= 0:
        print("Deposit amount must be positive")
        return
    self.__balance += amount

def withdrawal(self, amount):
    if amount <= 0:
        print("Withdrawal amount must be positive")
        return False
    if amount >= self.__balance:
        print("Insufficient balance")
        return False
    self.__balance -= amount
    return True
```

Never trust raw input — the method is the last line of defence before the data gets modified.

---

## Quick Recall Cheat Sheet

| Concept | What it means | Python example |
|---|---|---|
| Attribute | Data the object stores | `self.__balance` |
| Private attribute | Hidden from outside | `self.__name` (double underscore) |
| Getter | Method to read private data | `def getBalance(self):` |
| Setter | Method to update private data | `def setName(self, name):` |
| `__init__` | Where all attributes are initialized | `def __init__(self, name, bal):` |
| Encapsulation | Private data + public methods | `__balance` + `getBalance()` |
| Validation in methods | Guard data before modifying | `if amount >= self.__balance:` |
