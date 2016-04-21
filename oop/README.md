# SOLID

The term solid was coined by Robert C. Martin (aka Uncle Bob). Each letter stands for a strategy to write clean code:

## Single Responsiblity Principle

Every class or function that you write should have only one purpose. If it has multiple purposes think of a way to split it up into multiple pieces. The reasoning behind this is to create code that is easier to test in isolation. The code will also be more modular and sharable across projects.

Link: [Thoughtbot](https://robots.thoughtbot.com/back-to-basics-solid)

## Open Closed Principle

A class or function should be open for extension but closed for modification. To be more specific - in some cases you want to change the behavior of a system without changing every class. In many cases the strategy pattern can be used to archive such a goal.

[Thoughtbot](https://robots.thoughtbot.com/back-to-basics-solid)

## Liskov Substitution Principle

A class is Liskov substitutable if you can replace an instance of an class with a sublass without creating errors in the system. For a good example of a Liskov violation take a look at this post: [Thoughtbot](https://robots.thoughtbot.com/back-to-basics-solid)

## Interface Segregation Principle

This principle is less relevant in dynamic languages. Since duck typing languages don’t require that types be specified in our code this principle can’t be violated.

But in typed language try to minimize the interface that a class is using. A good example in Swift can be seen at this post:
[Thoughtbot](https://robots.thoughtbot.com/back-to-basics-solid)

## Dependency Inversion Principle

The Dependency Inversion Principle has to do with high-level (think business logic) objects not depending on low-level (think database querying and IO) implementation details.

[Thoughtbot](https://robots.thoughtbot.com/back-to-basics-solid)

# Command Query Separation

A method should either be a command (mutate state or object) or a query (read only). Try to never mix both.

# Composition vs. Inheritance

Only use inheritance when you have real abstractions. Sometimes inheritance is used to hide shared code but you're violating the SRP. Prefer composition over inheritance for these cases.

[Good Read](https://www.thoughtworks.com/de/insights/blog/composition-vs-inheritance-how-choose)

# Mutable vs. Immutable Datastructures

In most programming languages we used we have mutable datastructures. Mutating data can be very error prone (a thread is fucking up the data while another thread is needing it).

A great example for languages that embrace immutable data structures is closure (that use software transactional memory) or elm.

# Validate & Normalize at the boundaries

Always validate your user input. But where do you want such kind of code? In your models? No - you probably don't want to sprinkle your precious model code with verbose sanity checks. This is why you have to identify your boundaries where user input hits your model code. In Rails this is what strong parameters is all about. The params are the boundaries.
