---
layout: post
title: 'Building your DI Container'
date:   2026-01-08 12:00:00
categories:
  - csharp
---

Most developers that programs in OOP languages uses DI everyday but they dont understand why it is been used or what actually happens under the hood, before we dive deeper into DI. 
we need to understand inheritance (for base and derived classes) and interfaces in OOP.

Class Inheritance is a mechanism where one class (the Child) adopts the internal logic, data, and behavior of another class (the Parent). 
It creates a rigid "Is-A" hierarchy.

When you use inheritance, you are essentially saying, 
"This new thing is a specialized version of that existing thing." 
It allows you to write code once in a base class and have it automatically available in all derived classes, ensuring that they share the same "DNA."

```c#
// Base Class (Parent)
public class Vehicle 
{
    public int Speed { get; set; }
    
    public void Honk() 
    {
        Console.WriteLine("Beep beep!");
    }
}

// Derived Class (Child)
public class Car : Vehicle 
{
    public int DoorCount { get; set; }
}
```


An interface is essentially a blueprint or a "contract" that defines a set of related functionalities.
When a class or a struct implements an interface, it is making a promise to provide the implementation for all the members defined in that interface.

Basically you want implement Logging system, you would want to either log to a file, to a database or redis. you would have something like 

```c#
//Interface
public interface ILogger
{
  void Log(string message);
}

Now every implementation class will be 

public class DatabaseLogger : ILogger
{
  public void Log(string message)
  {
     //Logic goes here
  }
}


public class FileLogger : ILogger
{
  public void Log(string message)
  {
     //Logic goes here
  }
}

public class RedisLogger : ILogger
{
  public void Log(string message)
  {
     //Logic goes here
  }
}

```

##### Scenario 
Now say you have a huge codebase with thousand of files and at least the logging system was implemented in each file. 
If you are probably logging to the database for now and because of perfomance issue your app is facing , you decide to log to a file, 
it means you need make changes to each file from DatabaseLogger to FileLogger, 
you might probably make update to some files and not to some files. Not your fault, it a tedious thing to do. This is where DI comes in. 
Every file would have a service class (in this case interface or base class for inheritance) ILogger 
but the implementation would be done in just one file, say your Startup.cs/ Program.cs file


```markdown
container.Register<ILogger, FileLogger>();
```

If we decide change to log to redis, i just need to update the one line

```markdown
container.Register<ILogger, RedisLogger>();
```

and everything works as expected


In this post, we’ll build a **minimal but functional DI container** from scratch and learn how real frameworks like Microsoft.Extensions.DependencyInjection work internally.

By the end, you’ll understand:

- How object graphs are built
    
- How constructor injection works
    
- How lifetimes like **Transient**, **Scoped**, and **Singleton** are managed

What is a DI Container?

A Dependency Injection (DI) container is simply an object that:

- Knows **how to create** types
    
- Knows **what to inject** into them
    
- Controls **object lifetimes**

Instead of this:

```markdown
var service = new OrderService(new SqlRepository());
```

You write:

```markdown
var service = container.Resolve<IOrderService>();
```

Step 1: The Simplest Possible Container

We’ll start with a very basic container that uses reflection.

`public class SimpleContainer`
`{`
    `public T Resolve<T>() => (T)Resolve(typeof(T));`

    `private object Resolve(Type type)`
    `{`
        `var ctor = type.GetConstructors().Single();`
        `var parameters = ctor.GetParameters();`

        `var args = parameters`
            `.Select(p => Resolve(p.ParameterType))`
            `.ToArray();`

        `return Activator.CreateInstance(type, args)!;`
    `}`
`}`

## Step 2: Add Registrations (Interfaces → Implementations)

Now let’s make it useful.

`public class DiContainer`
`{`
    `private readonly Dictionary<Type, Type> _registrations = new();`

    `public void Register<TService, TImpl>()`
        `where TImpl : TService`
    `{`
        `_registrations[typeof(TService)] = typeof(TImpl);`
    `}`

    `public T Resolve<T>() => (T)Resolve(typeof(T));`

    `private object Resolve(Type type)`
    `{`
        `if (_registrations.TryGetValue(type, out var implType))`
            `type = implType;`

        `var ctor = type.GetConstructors().Single();`
        `var parameters = ctor.GetParameters();`

        `var args = parameters`
            `.Select(p => Resolve(p.ParameterType))`
            `.ToArray();`

        `return Activator.CreateInstance(type, args)!;`
    `}`
`}`

Step 3: Add Lifetimes (Transient & Singleton)

Let’s support real-world behavior.

public enum Lifetime
{
    Transient,
    Singleton
}

public class ServiceDescriptor
{
    public Type ServiceType { get; set; } = null!;
    public Type ImplType { get; set; } = null!;
    public Lifetime Lifetime { get; set; }
    public object? Instance { get; set; }
}

Container with lifetime support:

`public class Container`
`{`
    `private readonly Dictionary<Type, ServiceDescriptor> _services = new();`

    `public void Register<TService, TImpl>(Lifetime lifetime)`
    `{`
        `_services[typeof(TService)] = new ServiceDescriptor`
        `{`
            `ServiceType = typeof(TService),`
            `ImplType = typeof(TImpl),`
            `Lifetime = lifetime`
        `};`
    `}`

    `public T Resolve<T>() => (T)Resolve(typeof(T));`

    `private object Resolve(Type type)`
    `{`
        `if (!_services.TryGetValue(type, out var desc))`
            `desc = new ServiceDescriptor`
            `{`
                `ServiceType = type,`
                `ImplType = type,`
                `Lifetime = Lifetime.Transient`
            `};`

        `if (desc.Lifetime == Lifetime.Singleton && desc.Instance != null)`
            `return desc.Instance;`

        `var ctor = desc.ImplType.GetConstructors().Single();`
        `var args = ctor`
            `.GetParameters()`
            `.Select(p => Resolve(p.ParameterType))`
            `.ToArray();`

        `var obj = Activator.CreateInstance(desc.ImplType, args)!;`

        `if (desc.Lifetime == Lifetime.Singleton)`
            `desc.Instance = obj;`

        `return obj;`
    `}`
`}`


Usage:

```markdown
container.Register<IRepository, SqlRepository>(Lifetime.Singleton);
container.Register<IOrderService, OrderService>(Lifetime.Transient);
```

Adding Scoped Lifetimes to Our DI Container

A **scoped lifetime** means:

> “One instance per logical operation (usually per web request).”

- **Transient** → New instance every time
    
- **Singleton** → One instance for the entire app
    
- **Scoped** → One instance per scope
    

In ASP.NET Core, a scope usually maps to a **HTTP request** via IServiceScopeFactory.

Let’s implement the same concept.

Step 1 — Extend the Lifetime Enum

`public enum Lifetime`
`{`
    `Transient,`
    `Singleton,`
    `Scoped`
`}`

## Step 2 — Create a Scope Object

A scope holds instances that should live only inside that scope.

`public class Scope`
`{`
    `private readonly Dictionary<Type, object> _scopedInstances = new();`

    `public object? Get(Type type)`
        `=> _scopedInstances.TryGetValue(type, out var instance)` 
            `? instance` 
            `: null;`

    `public void Set(Type type, object instance)`
        `=> _scopedInstances[type] = instance;`
`}`

## Step 3 — Modify the Container to Support Scopes

We’ll split resolution into two layers:

- Root container → holds singletons
    
- Scope → holds scoped instances


`public class Container`
`{`
    `private readonly Dictionary<Type, ServiceDescriptor> _services = new();`
    `private readonly Dictionary<Type, object> _singletons = new();`

    `public void Register<TService, TImpl>(Lifetime lifetime)`
    `{`
        `_services[typeof(TService)] = new ServiceDescriptor`
        `{`
            `ServiceType = typeof(TService),`
            `ImplType = typeof(TImpl),`
            `Lifetime = lifetime`
        `};`
    `}`

    `public Scope CreateScope() => new Scope();`

    `public T Resolve<T>(Scope? scope = null)`
        `=> (T)Resolve(typeof(T), scope);`

    `private object Resolve(Type type, Scope? scope)`
    `{`
        `if (!_services.TryGetValue(type, out var desc))`
            `desc = new ServiceDescriptor`
            `{`
                `ServiceType = type,`
                `ImplType = type,`
                `Lifetime = Lifetime.Transient`
            `};`

        `// Singleton handling`
        `if (desc.Lifetime == Lifetime.Singleton)`
        `{`
            `if (_singletons.TryGetValue(type, out var instance))`
                `return instance;`

            `var created = CreateInstance(desc.ImplType, scope);`
            `_singletons[type] = created;`
            `return created;`
        `}`

        `// Scoped handling`
        `if (desc.Lifetime == Lifetime.Scoped)`
        `{`
            `if (scope == null)`
                `throw new InvalidOperationException("No active scope.");`

            `var existing = scope.Get(type);`
            `if (existing != null)`
                `return existing;`

            `var created = CreateInstance(desc.ImplType, scope);`
            `scope.Set(type, created);`
            `return created;`
        `}`

        `// Transient`
        `return CreateInstance(desc.ImplType, scope);`
    `}`

    `private object CreateInstance(Type type, Scope? scope)`
    `{`
        `var ctor = type.GetConstructors().Single();`
        `var args = ctor.GetParameters()`
            `.Select(p => Resolve(p.ParameterType, scope))`
            `.ToArray();`

        `return Activator.CreateInstance(type, args)!;`
    `}`
`}`

Step 4 — Example Usage of Scoped Services


`container.Register<IRepository, Repository>(Lifetime.Scoped);`
`container.Register<IOrderService, OrderService>(Lifetime.Transient);`

`var scope1 = container.CreateScope();`
`var svc1a = container.Resolve<IRepository>(scope1);`
`var svc1b = container.Resolve<IRepository>(scope1);`

`var scope2 = container.CreateScope();`
`var svc2  = container.Resolve<IRepository>(scope2);`

`// Same within a scope → true`
`Console.WriteLine(object.ReferenceEquals(svc1a, svc1b));`

`// Different across scopes → false`
`Console.WriteLine(object.ReferenceEquals(svc1a, svc2));`

## How This Maps to ASP.NET Core

ASP.NET Core uses the exact same idea internally:

- Scope = HTTP request
    
- Singletons = App lifetime
    
- Transients = Always new
    

This is what happens under the hood of Microsoft.Extensions.DependencyInjection.



There are more to DI such that this post does not address.

Other Injection method (method, property)
Circular dependency detect
Using Expression trees instead of reflection which is slow
Open generics
Code generation for performance (instead of slow reflection)