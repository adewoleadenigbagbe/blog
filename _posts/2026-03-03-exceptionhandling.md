---
layout: post
title: 'Exception Handling Best Practices'
date: 2026-03-03 12:00:00
categories:
  - csharp
---

Exception handling is a **correctness, reliability, and maintainability** concern—not just an error-handling mechanism. In C#, poor exception practices lead to hidden bugs, performance issues, and fragile systems. This guide focuses on **practical, real-world best practices** for writing robust C# code.


### 1. Catch Specific Exceptions, Not Exception
Catching `Exception` hides intent and makes recovery logic unreliable.Only catch exceptions you are expecting and can handle meaningfully.This allows code higher up the call stack to handle exceptions it can recover from or that are truly unexpected.

❌ **Bad**
```c#
try

{
    ProcessFile();
}

catch (Exception ex)

{
    Log(ex);
}
```

✅ **Good**

```c#
try

{
    ProcessFile();
}

catch (IOException ex)

{
    Log(ex);
}

catch (UnauthorizedAccessException ex)

{
    Log(ex);
}
```

### 2. Never Swallow Exceptions
The use of empty catch blocks are one of dangerous anti-patterns

❌ **Bad**

```c#

try

{
    DoWork();
}

catch
{
}
```

✅ **Good**

```c#

try

{
    DoWork();
}

catch (Exception ex)
{
    Log(ex);
    throw;
}
```

### 3. Re-throwing Exceptions

If you catch an exception to log it or add context, but you still need it to propagate, always use the simple throw; statement. **throw ex**  resets the stack trace to the point of the re-throw, obscuring the original source of the error. throw; preserves the original stack trace, which is critical for debugging.

❌ **Bad**

```c#

try

{
    DoWork();
}

catch
{
    throw ex;
}
```

✅ **Good**

```c#

try
{
    DoWork();
}

catch (Exception ex)
{
    throw;
}
```

### 4. Create Domain-Specific Exceptions

Define your own exception types when the error is specific to your application's business domain (e.g., InsufficientFundsException, UserNotFoundException). This allows callers to handle specific application errors precisely.

```c#
public class PaymentFailedException : Exception

{

    public PaymentFailedException(string message) : base(message) {}

}
```


### 5. Don’t Use Exceptions for Validation

Exception should not be returned for the user input, instead opt for validations. Reserve exceptions for **programming errors**, not user input.

❌ **Bad**
```c#
if (age < 0)

    throw new ArgumentException();

```

✅ **Good**
```c#
if (age < 0)

    return Result.Fail("Age must be positive");
```

### 6. Use Built-In Exception Types where Possible

Always use the builtIn exception there is in c#

- `ArgumentNullException`

- `ArgumentOutOfRangeException`

- `InvalidOperationException`

- `NotSupportedException`


### 7. Avoid Exceptions for Normal Control Flow

Exceptions are computationally expensive and should only be used for truly exceptional error conditions, not to control the flow of a routine operation.

❌ **Bad**

```c#
try {
    var result = dictionary["key"];
} catch (KeyNotFoundException) {
    // Handle not found
}
```

✅ **Good**

```c#
if (dictionary.TryGetValue("key", out var result)) {
    // Use result
} else {
    // Handle not found
}

```


### 8. Use finally Blocks for Cleanup

The code inside a finally block is guaranteed to execute, regardless of whether an exception occurred or not. Use it for essential cleanup operations that must run, such as closing a database connection or a file stream.
```c#

FileStream stream = null;

try

{

    stream = OpenFile();

}

finally

{

    stream?.Dispose();

}

//OR
using var stream = OpenFile();

```

### 9. Centralized Handling
Implement a Global/Top-Level Handler. In larger applications (like ASP.NET Core APIs or desktop apps), implement a single, centralized mechanism (e.g., middleware, a global event handler) to catch and log any unhandled exceptions before the application crashes. This ensures that no error is entirely missed.

✅ **Good**
```c#
app.UseExceptionHandler("/error");

//OR

app.Use(async (ctx, next) =>

{

    try

    {

        await next();

    }

    catch (Exception ex)

    {

        Log(ex);

        throw;

    }

});
```

### 10. Always log **context**, not just stack traces.

```c#
Log.Error(ex, "Failed to process order {OrderId}", orderId);
```