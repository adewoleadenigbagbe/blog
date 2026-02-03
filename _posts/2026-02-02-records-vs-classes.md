---
layout: post
title: 'Records vs Classes'
date: 2026-02-02 12:00:00
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

A **record** is a reference type designed to represent immutable data, where value-based equality matters more than identity. Equality is based on the values not references. They are ideal for DTOs, messages, request/responses and configs.Below is how you declare a **record** in C#

``` csharp

public record Product(Guid Id, string Name);

```

### Equality in Records
``` csharp

var id = Guid.NewGuid();

var product1 = new Product(id, "Toy");
var product2 = new Product(id, "Toy");

Console.WriteLine(product1 == product2); // true

var product3 = new Product(Guid.NewGuid(), "Toy");
Console.WriteLine("Are both instance of the record equal : {0}", product2 == product3); // return false

```

Before records was introduced in c# 9.0, fields equality comparism look like this with classes

``` csharp

public class User
{
    public string Name { get; init; }

    public int Age { get; init; }

    public override bool Equals(object? obj)
    {
        if(obj is not User other) 
        {
          return false;
        }

        return Name == other.Name && Age == other.Age;
    }

    public override int GetHashCode() => HashCode.Combine(Name, Age);
}

```

Records are immutable by default but easy to copy using **with** keyword. This is **huge** for Thread safety , predictable state

``` csharp

var updatedUser = user with { Name = "Alice" };

```

We do have record allocated on the heap (record class) which i discussed above and we also have record allocated on the stack (record struct)
``` csharp

// Record class - Use cases: Object size is moderate - Passed across layers - Used in APIs and DTOs
public record Product(string Name);

//Record Struct - Use cases: Small, immutable value objects - High-performance
public record struct Money(decimal Amount, string Currency);

```

### Difference between Types

| Type          | Allocation |  Value Equality | Immutability         |
|---------------|------------|---------------|----------------------|
|     class     |    Heap    | No (by default) |          No          |
|     record    |    Heap    |       Yes       |          Yes         |
|     struct    |    Stack   |       Yes       |          No          |
| record struct |    Stack   |       Yes       | No (yes if readonly) |

*Records don't replace classes --- they complete the type system.* Now lets discuss the use cases of both record and classes. What you should use them for and what you shouldnt use them for.

### Use **record** when:

-   DTOs

-   API contracts (Request/Response)

-   Messages/events

-   Configuration objects

-   LINQ projections

-   Value objects

### Use **class** when:

-   Domain entities (EF Entities)

-   Services

-   Controllers

-   Aggregates

-   Stateful components

### Use **record struct** when:

-   Small immutable values

-   Performance-critical paths

-   Financial or math domains

### ✅ Example - DTOs & Linq Projections and Value Equality (Perfect Use Case)

``` csharp

public record OrderDto(Guid Id, decimal Total);

var orders = context.Orders

    .Select(o => new OrderDto(o.Id, o.Total))

    .ToList();

var users = people.Select(p => new UserDto(p.Name, p.Email)).Distinct();

// Value equality makes: - `Distinct()` - `GroupBy()` - `HashSet<T>` work correctly **without extra code**.

```


## Records with Pattern matching


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

// This style: - Eliminates null checks - Avoids invalid states - Improves correctness

```

## Final Rule of Thumb

> **Behavior → Class**

> **Data → Record**

> **Small immutable value → record struct**


You can get code samples for this post here [Records vs Classes](https://github.com/adewoleadenigbagbe/Blog-Code-Samples/tree/main/RecordsVsClasses)