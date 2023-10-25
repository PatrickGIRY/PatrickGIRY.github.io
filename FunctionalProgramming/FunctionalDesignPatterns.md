# Functional Design Patterns

Things FP developers do a lot.

## OO pattern/principe

* Single Responsibility Principle
* Open/Close Principle
* Dependency Inversion Principle
* Interface Segregation Principle
* Factory pattern
* Strategy pattern
* Decorator pattern
* Visitor pattern

## Core Principles of FP design

### Function

    apple -> banana

A function is a standalone thing not attached to a class.

But in Java everything have to be in a class.

A method is a way to represent function in Java.

A method can be functional if it respects the requirements of a pure function:

* It must not mutate anything outside the function. No internal mutation may be visible from the outside.
* It must not mutate its argument.
* It must not throw errors or exceptions.
* It must always return a value.
* When called with the same argument, it must always return the same result.

## Function as value

With the lambda the function can be manipulate as a value.

    var z = 1;

    var add = (x, y) -> x + y;

## Function type is an interface

Let's take the Single Responsibility Principle and the Interface Segregation Principle to the extreme.
Every interface should have only one abstract method.

## Function as output

    interface Matcher {
        Predicate<String> match(String query);
    }

    var add = x -> y -> x + y;
