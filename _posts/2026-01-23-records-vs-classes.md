---
layout: post
title: 'Records vs Classes. A Complete Guide (With EF Core, Performance, and Real-World Usage)'
date: 2026-01-23 12:00:00
categories:
  - csharp
---

C# records were introduced to solve a long-standing problem: **modeling data without boilerplate and without accidental mutation**.They are not a replacement for classes.They are a *different modeling tool*.

In this article covers everything you need to know to confidently choose between **records, classes, structs, and record structs** in real-world C# systems.

## Classes: Identity + Behavior

A **class** represents an object that has **identity**, changes **state** over time and contains **behavior**. A standard class contain the following

- Fields - for class Internals 👉 (storage), 
- Properties - controlled data access 👉 (Exposure)
- Methods - 👉 Behaviour and actions
- Constructors - How a class is created 👉 (setup)
- Events - 👉 Notification System
- Constants - 👉Fixed never changed
- Readonlys - 👉 Set Once , then frozen
- Nested Class - 👉 Classes inside classes

``` csharp
public class User
{
    private int _count;

    public string Name { get; set; }
    
    public void SetName(string name)
    {
      // does an action here
    }

    public User(string name)
    {
       Name = name;
    }

    public const int MaxRetry = 3;

    public readonly Guid Id = Guid.NewGuid();
}
```

You use methods in the class to manipulate fields and properties in that way you dont have a new copy of the data when it changes.If you make two instances of the same classes, they are not going to be equal
because 
``` csharp

var u1 = new User { Name = "John" };

var u2 = new User { Name = "John" };

Console.WriteLine(u1 == u2); // false

var user3 = user2;
Console.WriteLine("Are both instance of the class equal : {0}", user2 == user3); // return true

```
 
## Records: Data + Value Semantics

A **record** represents **data**, not identity.

``` csharp

public record User(string Name);

```

### Equality in Records
``` csharp

var u1 = new User("John");

var u2 = new User("John");

Console.WriteLine(u1 == u2); // true

```
Records use **value-based equality** by default.

## 3. Why Records Exist (The Problem They Solve)

Before records, DTOs looked like this:

``` csharp

public class UserDto

{
    public string Name { get; init; }

    public override bool Equals(object obj) { ... }

    public override int GetHashCode() { ... }
}

```

**Records generate all of this for you**: - Equality - Hash codes -

`ToString()` - Copy semantics
  

## 4. Non-Destructive Mutation (`with`)

Records are immutable by default --- but easy to copy.

``` csharp

var updated = user with { Name = "Alice" };

```

This is **huge** for: - Thread safety - Predictable state -

Functional-style programming

## 5. `record class` vs `record struct` vs `struct`

### record class (default)

-   Reference type

-   Heap allocated

-   Value equality

``` csharp

public record Product(string Name);

```
**Use when**: - Object size is moderate - Passed across layers - Used in

APIs and DTOs

### struct

-   Value type

-   Stack allocated (mostly)

-   Mutable by default

-   No inheritance

``` csharp

public struct Point
{

    public int X;

    public int Y;
}

```

**Use when**: - Very small data - Performance-critical - No mutation

pitfalls


### record struct

-   Value type + value equality + immutability

``` csharp

public readonly record struct Money(decimal Amount, string Currency);

```

**Best for**: - Small, immutable value objects - High-performance

domains - Financial and math models

------------------------------------------------------------------------

## 6. EF Core: Records vs Classes

### ❌ Entity Models (Avoid Records)

EF Core **expects mutable entities**.

Problems with records: - Change tracking issues - Proxy generation -

Required parameterless constructors

``` csharp

// ❌ Not recommended

public record Order(Guid Id, decimal Total);

```
------------------------------------------------------------------------

### ✅ DTOs & Projections (Perfect Use Case)

``` csharp

public record OrderDto(Guid Id, decimal Total);

var orders = context.Orders

    .Select(o => new OrderDto(o.Id, o.Total))

    .ToList();

```

**Rule**: - **Entities → Class** - **DTOs → Record**

## 7. Records and LINQ (Where They Shine)

Records are ideal for LINQ projections:

``` csharp

var users = people

    .Select(p => new UserDto(p.Name, p.Email))

    .Distinct();

```

Value equality makes: - `Distinct()` - `GroupBy()` - `HashSet<T>`

work correctly **without extra code**.


## 8. Performance Considerations

### Allocation

  

  Type            Allocation

  --------------- ------------

  class           Heap

  record class    Heap

  struct          Stack

  record struct   Stack

### Equality Cost

-   Records generate optimized `Equals`

-   Often **faster and safer** than handwritten overrides

### Copy Cost

-   `with` creates a new instance

-   Fine for DTOs

-   Avoid in tight loops for large objects

## 9. Records in Functional-Style C

Records pair beautifully with: - Pattern matching - Immutability -

Expressions

``` csharp

public record OrderState;

public record Pending : OrderState;

public record Paid : OrderState;

string Describe(OrderState state) => state switch

{

    Pending => "Waiting for payment",

    Paid => "Payment completed",

    _ => "Unknown"

};

```
This style: - Eliminates null checks - Avoids invalid states - Improves correctness


## 10. When NOT to Use Records

Avoid records when: - Identity matters - State changes frequently - You rely on ORMs for tracking - Object lifecycle is complex

``` csharp

public class BankAccount
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount) { ... }
}

```

This should **never** be a record.



## 11. Practical Decision Guide

### Use **record** when:

-   DTOs

-   API contracts

-   Messages/events

-   Configuration objects

-   LINQ projections

-   Value objects

### Use **class** when:
-   Domain entities

-   Services

-   Controllers

-   Aggregates

-   Stateful components

### Use **record struct** when:

-   Small immutable values

-   Performance-critical paths

-   Financial or math domains


## 12. Final Rule of Thumb

> **Behavior → Class**

> **Data → Record**

> **Small immutable value → record struct**


*Records don't replace classes --- they complete the type system.*