Hey! I am writing this list because I know too many programming langudges and dunno what to choose. I include the following:

- Common uses
- Tech area
- Detailed description
- Things I hate/love
- Syntax
- Popularity

Note some languages are excluded because a better no-cost version exists (looking at you JavaScript). Typically, this means that someone hiring you for a language would be fine if you programmed in the "better" version. They will be indicated like:
- ~~JavaScript~~ TypeScript

To prevent bias, Languages are ordered alphabetically.

Enjoy! 

## Actual list

- C++
- C#
- C
- Crystal
- D
- Dart
- Erlang
- Elm
- Elixer
- Fortran
- Groovy
- Nim
- Python
- Rust
- Raku
- Ruby
- R
- Swift
- Java
- JavaScript
- TypeScript
- Go
- Perl
- PureScript
- Haskell
- Kotlin
- PHP
- Scala
- Julia
- Zig
- lisp
	- Clojure
	- [Racket](https://en.wikipedia.org/wiki/Racket_(programming_language))
	- Carp
- Fennel
- Red
- Porth

## Removing the nonos

### Duplicates

- Typescript is objectively better than JavaScript in every single way, so ~~JavaScript~~
- Raku is probably better than ~~Perl~
	- But it confuses me
- Kotlin is certainly a better option than Java

```
| Feature                     | Kotlin                                                                                                                                                                                                                                              | Java                                                                                                                                                                                                                                                                |   |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---|
| Extension Functions         | The extension function is available in Kotlin. Android extensions are useful as they allow developers to add methods to classes without making changes to their source code. Developers have the ability to extend a class with new functionality.  | Java developers need to create a new class to extend the functionality of the existing class, so from now users can use a newly created class everywhere to use the extended functionality.                                                                         |   |
| Null Safety                 | Available. Kotlin’s type of system has inbuilt null safety.                                                                                                                                                                                         | Not available. In fact, NullPointerException is mainly responsible for the development errors of Android and Java.                                                                                                                                                  |   |
| Static Members              | In Kotlin, we can use the companion object to create static members of the class.  An object declaration inside a class can be marked with the companion keyword.                                                                                   | Available in Java. It is used for memory management mainly. One can apply java static keywords with variables, methods, blocks, and nested classes.                                                                                                                 |   |
| String Templates            | Yes, there are two types of string literals in Kotlin such as escaped string and raw string.  Kotlin string template also supports expression.                                                                                                      | Available in Java too, but it doesn’t support expressions like Kotlin.                                                                                                                                                                                              |   |
| Coroutines                  | In Kotlin, coroutines are concurrency design patterns. It can be used to simplify code on Android that performs asynchronously. Coroutines were added to Kotlin in the 1.3 version and they are based on established concepts from other languages. | In Java, users have two different options, such as RxJava and Project Loom. RxJava is a library for composing asynchronous and event-based programs using observable sequences, and Project Loom supports a high-throughput, lightweight concurrency model in Java. |   |
| Wildcard-types              | No, Kotlin doesn’t have any wildcard-types. But it has two other things, including declaration-site variance and type projections.                                                                                                                  | Available in Java. The wildcard, in general, code means (?) that represents an unknown type.  It can be used in different situations.                                                                                                                               |   |
| Smart Casts                 | Yes, this feature is available in Kotlin. It helps the Kotlin compiler to track conditions inside of the expression. If the compiler finds a variable that is not null of type nullable, the compiler allows accessing the variable                 | No, it is not available in Java. However, to know the types in Java, we can use something like an instance to check the type and then cast it to the right type.                                                                                                    |   |
| Lazy Keyword                | Yes, it is available in Kotlin. It mainly decreased startup time that is useful when using Kotlin for app development.                                                                                                                              | This feature is not available in Java.                                                                                                                                                                                                                              |   |
| Operator Overloading        | Yes, Kotlin allows users to provide a way to invoke functions.  It allows us to do an arithmetic operation, equality checks, or compare whatever object we want through symbols like +, -, /, *, %, <, >.                                           | In Java, operators are tied to particular Java types. For instance, String and numeric types in Java can make use of the + operator for concatenation and addition. Another Java type can’t reuse this operator.                                                    |   |
| Blend of Functional and OOP | Kotlin is the blend of functional and object oriented programming.                                                                                                                                                                                  | Java is based only on OOP (Object Oriented Programming).                                                                                                                                                                                                            |   |
| Language Scripting          | Kotlin offers language scripting capabilities to use the language straight into your Gradle build scripts.                                                                                                                                          | Java does not enable programmers to use language scripting capabilities.                                                                                                                                                                                            |   |
| Lambda Expression           | Lambda Expression is supported in Kotlin.                                                                                                                                                                                                           | The Lambda Expression is not supported in Java.                                                                                                                                                                                                                     |   |
```
## V2 

a bit of sorting

- C#
- C
	- C++
- Clojure
- D
- Dart
- Erlang
- Elixer
- Fortran
- Groovy
- Nim
- Python
- Rust
- R
- Ruby
	- Crystal
- Swift
- ~~Java~~ Kotlin
- ~~JavaScript~~ TypeScript
- Go
- ~~Perl~~ Raku
- PureScript
- Haskell
- Scala
- Julia
- Racket
- Zig