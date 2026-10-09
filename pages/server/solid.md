# SOLID Principles

## Definition

**SOLID** is a set of five object-oriented design principles that help you write code that is easier to understand, maintain, extend, and test. The acronym was popularized by Robert C. Martin (Uncle Bob).

| Letter | Principle | One-line meaning |
|--------|-----------|------------------|
| **S** | Single Responsibility Principle (SRP) | A class should have only one reason to change |
| **O** | Open/Closed Principle (OCP) | Open for extension, closed for modification |
| **L** | Liskov Substitution Principle (LSP) | Subclasses must be usable in place of their parent class |
| **I** | Interface Segregation Principle (ISP) | Don't force classes to depend on methods they don't use |
| **D** | Dependency Inversion Principle (DIP) | Depend on abstractions, not concrete implementations |

---

## 1. Single Responsibility Principle (SRP)

**Definition:** A class should do one job only, so there is only one reason to change it.

**Bad:**

```python
class Invoice:
    def calculate_total(self): ...
    def save_to_database(self): ...
    def print_invoice(self): ...
```

This class handles business logic, storage, and printing.

**Good:**

```python
class Invoice:
    def calculate_total(self): ...

class InvoiceRepository:
    def save(self, invoice): ...

class InvoicePrinter:
    def print(self, invoice): ...
```

---

## 2. Open/Closed Principle (OCP)

**Definition:** You should be able to add new behavior without changing existing code.

**Bad:**

```python
class DiscountCalculator:
    def calculate(self, customer_type, amount):
        if customer_type == "regular":
            return amount * 0.95
        elif customer_type == "premium":
            return amount * 0.90
        # every new type requires editing this method
```

**Good:**

```python
from abc import ABC, abstractmethod

class Discount(ABC):
    @abstractmethod
    def apply(self, amount): ...

class RegularDiscount(Discount):
    def apply(self, amount):
        return amount * 0.95

class PremiumDiscount(Discount):
    def apply(self, amount):
        return amount * 0.90

def calculate(discount: Discount, amount):
    return discount.apply(amount)
```

Adding a new discount type means adding a new class, not editing old code.

---

## 3. Liskov Substitution Principle (LSP)

**Definition:** If `B` is a subclass of `A`, you should be able to use `B` anywhere `A` is expected without breaking the program.

**Bad:**

```python
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly")  # breaks expectations
```

**Good:**

```python
class Bird:
    def eat(self): ...

class FlyingBird(Bird):
    def fly(self): ...

class Sparrow(FlyingBird): ...
class Penguin(Bird): ...
```

---

## 4. Interface Segregation Principle (ISP)

**Definition:** Prefer several small, specific interfaces over one large, general one.

**Bad:**

```python
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...

class Robot(Worker):
    def work(self): ...
    def eat(self):
        pass  # robots don't eat
```

**Good:**

```python
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Human(Workable, Eatable): ...
class Robot(Workable): ...
```

---

## 5. Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Bad:**

```python
class MySQLDatabase:
    def save(self, data): ...

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # tightly coupled
```

**Good:**

```python
class Database(ABC):
    @abstractmethod
    def save(self, data): ...

class MySQLDatabase(Database):
    def save(self, data): ...

class MongoDatabase(Database):
    def save(self, data): ...

class UserService:
    def __init__(self, db: Database):
        self.db = db  # injected abstraction
```

Now you can swap databases (or use a mock in tests) without changing `UserService`.

---

## Benefits

- Easier to maintain and refactor
- Simpler unit testing
- Less coupling, more reusable code
- Safer to extend with new features

## Quick Memory Aid

> **S**ingle job, **O**pen to extend, **L**ike-for-like substitution, **I**nterfaces kept small, **D**epend on abstractions.