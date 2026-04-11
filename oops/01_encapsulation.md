## What is Encapsulation?

Encapsulation is an OOP principle of **bundling data (attributes) and the methods that operate on that data together inside a class**, while hiding internal details from the outside world. External code interacts with the object only through well-defined methods — not by directly touching its data.

Think of it like a `Library` system: the internal book records are managed privately. You don't directly reach into the system and change a book's status — you call `borrow_book()` or `return_book()`, which apply the right logic for you.

---

## Core Idea: Data Hiding + Controlled Access

<img width="1440" height="448" alt="image" src="https://github.com/user-attachments/assets/9be03f45-b353-416a-915c-ab298d91fc47" />
Private attributes are only accessible through public methods. This gives the class full control over how its data is read or modified.

## Access Modifiers in Python

Python uses **naming conventions** (not strict keywords) to indicate access levels:

| Convention | Example | Access level |
|---|---|---|
| `name` | `self.title` | Public — accessible anywhere |
| `_name` | `self._author` | Protected — convention only, still accessible |
| `__name` | `self.__is_available` | Private — name-mangled by Python |

> **Name mangling:** `self.__is_available` internally becomes `self._Book__is_available`. It's not truly inaccessible, but it signals "don't touch this directly."

---

## Real-World Example: Book & Library

### The `Book` class — encapsulates a single book's state

```python
class Book:
    def __init__(self, title, author):
        self.title = title          # public
        self.author = author        # public
        self.is_available = True    # public, managed via methods

    def borrow(self):
        if self.is_available:
            self.is_available = False
            return True
        return False                # already borrowed

    def return_book(self):
        self.is_available = True

    def __str__(self):
        status = "Available" if self.is_available else "Not Available"
        return f"{self.title} by {self.author} -> {status}"
```

`borrow()` and `return_book()` enforce the rules — you can never set `is_available` to an arbitrary value from outside without going through these methods.

---

### The `Library` class — encapsulates the book collection

```python
class Library:
    def __init__(self):
        self.books = {}  # key: title, value: Book object

    def add_book(self, title, author):
        if title in self.books:
            print("Book already exists.")
        else:
            self.books[title] = Book(title, author)
            print(f"{title} added to library.")

    def borrow_book(self, title):
        book = self.books.get(title)
        if not book:
            print("Book not found.")
            return
        if book.borrow():
            print(f"{title} borrowed successfully.")
        else:
            print("Book is not available.")

    def return_book(self, title):
        book = self.books.get(title)
        if not book:
            print("Book not found.")
            return
        book.return_book()
        print(f"{title} returned successfully.")

    def check_availability(self, title):
        book = self.books.get(title)
        if not book:
            print("Book not found.")
            return
        print(f"{title} availability:", "true" if book.is_available else "false")

    def display_books(self):
        if not self.books:
            print("Library is empty.")
            return
        print("\nLibrary Books:")
        for book in self.books.values():
            print(book)
```

Notice: the `Library` class never directly flips `book.is_available = True/False` — it always delegates to `book.borrow()` and `book.return_book()`. Each class manages its own state.

---

### Running the full flow

```python
library = Library()

# Add books
library.add_book("Python Basics", "Aju")
library.add_book("Java Fundamentals", "Hari")
library.add_book("Data Structures", "Neha")

library.display_books()

# Borrow flow
library.borrow_book("Python Basics")
library.borrow_book("Python Basics")  # already borrowed — blocked

library.check_availability("Python Basics")   # false

library.return_book("Python Basics")
library.check_availability("Python Basics")   # true

# Edge cases
library.borrow_book("C++")   # not found

library.display_books()
```

**Output:**
```
Python Basics added to library.
Java Fundamentals added to library.
Data Structures added to library.

Library Books:
Python Basics by Aju -> Available
Java Fundamentals by Hari -> Available
Data Structures by Neha -> Available

Python Basics borrowed successfully.
Book is not available.
Python Basics availability: false
Python Basics returned successfully.
Python Basics availability: true
Book not found.

Library Books:
Python Basics by Aju -> Available
Java Fundamentals by Hari -> Available
Data Structures by Neha -> Available
```

---

## Class Responsibilities at a Glance

<img width="1440" height="488" alt="image" src="https://github.com/user-attachments/assets/e271bd67-5b57-4370-8cb3-3c150a38aa69" />

Each class owns its own data and exposes only what is necessary. `Library` calls `Book`'s methods rather than directly touching its attributes — this is encapsulation working across two classes.

---

## Why Encapsulation Matters

**Data security** — internal state can only change through defined methods, preventing accidental or invalid modifications.

**Controlled logic** — methods like `borrow()` enforce rules (e.g. a book can't be borrowed twice) rather than letting callers set flags freely.

**Flexibility** — internal implementation can change (e.g. switching `is_available` from a boolean to a count for multiple copies) without breaking any code that uses the class, as long as the public methods stay the same.

**Modular and maintainable** — each class is self-contained. Debugging is focused: if borrow logic breaks, you look inside `Book`, not across the codebase.

---

## Key Takeaways for Interviews

- Encapsulation = bundling data + methods together + restricting direct access
- Python uses `__` (double underscore) for name-mangled private attributes — it is a convention, not a hard lock
- Getter and setter methods (or controlled action methods like `borrow()`) are the proper interface to private data
- Encapsulation is a *design principle*; access modifiers are the *mechanism* to achieve it
- Two classes can be individually encapsulated and still cooperate cleanly by calling each other's public methods — never poking into each other's internals directly
