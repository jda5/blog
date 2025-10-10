---
date:
  created: 2024-07-30
pin: true
links:
  - Homepage: index.md
categories:
  - Business
tags:
  - leadership
  - management
authors:
  - jacob
slug: what-we-know-about-leadership
---

# SOLID Design Principles

I recently embarked on a painful software refactor. Just a few months ago, when we began building our application, our codebase was clean and efficient. Every function was meticulously documented, and data flowed elegantly through our system.

But working in a nascent start-up has its own demands. The need to rapidly demonstrate a proof-of-concept and outperform competitors led to some quick and dirty solutions. A hack here, an anti-pattern there, and soon our ounce spotless codebase started to degrade. It became rigid and fragile. Minor changes triggered a cascade of subsequent changes, causing our application to break often. Design preserving methods became harder to write than hacks, and with that our codebase was well and truly rotting.

I decided I was going to do this refactor properly and re-read Mariano Anaya’s Clean Code in Python for some inspiration. It was there in Chapter 4 that I discovered the SOLID Principles.

SOLID is a mnemonic acronym for a set of five principles intended to make object-oriented codebases more maintainable, extendable and ultimately much easier to work with. It stands for:

- **S**ingle-Responsibility Principle
- **O**pen/Closed Principle
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

These principles were introduced by Robert C. Martin (aka. Uncle Bob) in Design Principles and Design Patterns as a means of preventing software degradation. Robert actually presents them in a slightly different order (OLDIS) and specifies that some principles are more important than others. In any case, in this post I will provide an overview of each principle (though in a slightly different order: OLSID), illustrating each with an example.

# Open/Closed Principle

*A module should be open for extension but closed for modification.*

Of the five principles, this is the most important. The basic idea is that we should be able to add new features without changing any existing code, simply by writing new code.

To illustrate this, let’s suppose we are designing a simple application that serves as a booking system for a restaurant and have the following code:

```py
from datetime import datetime

class Customer:

    def __init__(self, name: str, contact_details: dict, is_regular: bool):
        self.name = name
        self.contact_details = contact_details
        self.is_regular = is_regular

class Reservation:

    def __init__(self, customer: Customer, datetime: datetime) -> None:
        self.customer = customer
        self.datetime = datetime

    def notify_customer(self, preference):
        if preference == "email":
            # Contact the customer via email
            ...
        elif preference == "phone":
            # Contact the customer via phone
            ...
```

We first define a `Customer` class with a name, contact details (e.g., email, phone, etc.), and a Boolean indicating whether they are a regular customer.

The `Reservation` class depends on a `Customer` who made the booking and includes the date and time of the booking. Additionally, it has a `notify_customer` method that notifies the customer about updates to the reservation based on their preferred contact method (e.g., email, phone, etc.).

The problem here is that every time we want to add a new notification method – for example, “WhatsApp” – we need to make changes to the `notify_customer` function. This violates the Open/Closed Principle, as we should be able to add new features without changing our existing code.

Abstraction is the key to the Open/Closed Principle and can be achieved through static polymorphism, also known as interfaces (use of templates or generics). Let’s refactor our code to ensure we follow this principle.

```py
from datetime import datetime
from abc import ABC, abstractmethod

class Notifier(ABC):

    @abstractmethod
    def notify(self, info: str):
        # Notify the customer about their reservation
        ...

class EmailNotifier(Notifier):

    def notify(self, email: str):
        # Contact the customer via email
        ...

class PhoneNotifier(Notifier):

    def notify(self, number: str):
        # Contact the customer via phone
        ...

class Reservation:

    def __init__(self, customer: Customer, datetime: datetime) -> None:
        self.customer = customer
        self.datetime = datetime

    def notify_customer(self, notifier: Notifier): 
        notifier.notify(self.customer.contact_preference)
```

By creating a `Notifier` interface using Python’s abstract base class (ABC), we can now easily add new notification systems by creating a new subclass. Additionally, we no longer need to modify the notify_customer method to add this feature. For example, we can create a WhatsApp notification system as follows:

```py
class WhatsAppNotifier(Notifier):

    def notify(self, customer_id: int):
        # Contact the customer via phone
        ...
```

Some of you may have already spotted some problems with this implementation, which we will address as part of the next principle. While the Open/Closed Principle can be challenging to fully implement, even partial compliance can dramatically improve the structure of our application.

# Liskov Substitution Principle

*Subclasses should be substitutable for the base classes.*

This principle derives from the concept of Design by Contract by Bertrand Meyer. Classes that derive from a base class must honour the structure of that base class. In other words, software entities (classes, functions, etc.) that use a base class should continue to function properly if a derivative of that base class is passed to them.

The problem in our example above is that each subclass of the `Notifier` base class has different arguments. The `EmailNotifier` depends on an email argument (a string), whereas the `WhatsAppNotifier` depends on a customer ID (an integer). This inconsistency will invariably cause problems for our `notify_customer` method, as it cannot consistently use each derivative of the Notifier base class. Furthermore, the code will break completely when a notification system needs more than one argument (for example, the phone might require a country code as well as the base number).

How could we fix this?

Instead of passing the contact details, we should pass the `Customer` object itself. Then each method can fetch the information it needs directly from the customer.

```py
class Notifier(ABC):

    @abstractmethod
    def notify(self, customer: Customer):
        # Notify the customer about their reservation
        ...

class EmailNotifier(Notifier):

    def notify(self, customer: Customer):
        # Contact the customer via email
        ...
```

And so, our `notify_customer` method becomes:

```py
class Reservation:

    def __init__(self, customer: Customer, datetime: datetime) -> None:
        self.customer = customer
        self.datetime = datetime

    def notify_customer(self, notifier: Notifier): 
        notifier.notify(self.customer)
```

## Single-Responsibility Principle

*A class should have one and only one reason to change, meaning that a class should have only one job.*

Returning to the "S" in our SOLID principles, the Single-Responsibility Principle states that we want to increase cohesion among things that change for the same reason and decrease the coupling among things that change for different reasons. In other words, we aim to isolate modules and design systems that are responsible for just one function.

Our `notify_customer` method currently violates this principle. Notifying customers should be conceptually distinct from handling their reservations. Our current implementation couples reservations with notifications, causing problems when we want to test each system in isolation. We shouldn’t need to implement a notification system when writing unit tests for our Reservation class.
Let’s fix this…

```py
class Reservation:

    def __init__(self, customer: Customer, datetime: datetime) -> None:
        self.customer = customer
        self.datetime = datetime

def notify_customer(reservation: Reservation, notifier: Notifier):
    customer = reservation.customer
    notifier.notify(customer)
```

By creating a function separate from the `Reservation` class, we ensure that changes to the notification system do not affect the reservations.

In practice, this principle is a lot more subjective than the previous two, and you can certainly take it too far. Working with a system of hundreds of singleton interfaces sounds like hell. Ideally, you want to work with several small, conceptually distinct classes that are easy to manage and test, without having to spend ages navigating through your codebase to find what you’re looking for.

## Interface Segregation Principle

A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.

Jumping ahead to the "I" in our SOLID acronym, the Interface Segregation Principle means that we shouldn’t force methods or attributes onto subclasses that they don’t use. It is better to create many client-specific interfaces than a general-purpose monolith. Creating a specific instance for each client and using multiple inheritance is preferable to creating a base class with all the methods that clients need.

For instance, suppose we want to create a brand-new Employee class. We realise that an Employee has a name and contact details, so we decide to reuse some of the code we wrote for the Customer (which is generally a good thing).

```py
class Employee(Customer):

    def __init__(self, name: str, contact_details: dict):
        super().__init__(name, contact_details, False)
```

The problem here is that the Employee class is forced to implement the `is_regular` Boolean, which is not appropriate. Instead, we should create additional abstractions that both `Employee` and `Customer` can extend.

```py
class Person:

    def __init__(self, name: str, contact_details: dict) -> None:
        self.name = name
        self.contact_details = contact_details

class Employee(Person):
    ...

class Customer(Person):
    
    def __init__(self, name: str, contact_details: dict, is_regular: bool) -> None:
        super().__init__(name, contact_details)
        self.is_regular = is_regular
```

In the above implementation, we have reduced code repetition, meaning that we only need to make changes in one place when necessary. Additionally, our Employee class is no longer forced to inherit the is_regular attribute, which it does not use.

## Dependency Inversion Principle

Depend upon Abstractions. Do not depend upon concretions.

This final principle states that dependencies should always target an interface and never rely on a concrete implementation. This prevents us from depending on volatile software entities. 

Our restaurant reservation application already adheres to this principle. I think if you get the Liskov Substitution Principle right, the Dependency Inversion Principle naturally falls into place.

Nonetheless, to illustrate this principle, let’s suppose our restaurant has a special birthday party Reservation subclass and a method to cancel reservations.

```py
class BirthdayParty(Reservation):
    ...

def cancel_booking(booking: BirthdayParty):
    ...
```

The above violates this principle because any high-level entity (the cancel_booking method) must not depend on the low-level entity (`BirthdayParty`). It should instead depend on the abstraction (the `Reservation` class).

This conceptually makes sense since you should be able to cancel bookings in a generalised manner without knowing the specific type of reservation. By depending on the Reservation abstraction, the cancel_booking method becomes more flexible, capable of handling any type of reservation that extends the Reservation class, aka:

```py
def cancel_booking(booking: Reservation):
    ...
```

