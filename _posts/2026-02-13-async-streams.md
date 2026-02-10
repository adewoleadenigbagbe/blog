---
layout: post
title: 'Async Stream in C#'
date: 2026-02-13 12:00:00
categories:
  - csharp
---

An **async stream** is like a regular stream (`IEnumerable<T>`) but asynchronous. It allows you to produce and consume data **asynchronously**, this was a game-changer introduced in C# 8.0. Before this, if you wanted to stream data asynchronously, you often had to wait for the entire collection to be ready before you could do anything with it. It is perfect reading large datasets without blocking the thread, streaming data from remote sources like APIs, producing values over time without waiting for the full collection.It's the difference between waiting for a bucket to fill up with water before drinking versus drinking straight from a running tap.Async streams use `IAsyncEnumerable<T>` and `await foreach` instead of `IEnumerable<T>` and `foreach`.

The Problem with traditional Task<IEnumerable<T>> is that you have a Late Delivery problem which allow you to buffer data into memory. The entire task mush finish, the calling code has to await the entire Task before it gets access to the IEnumerable, also buffering is required: Because a Task only returns a single value once it's completed, you cannot "yield" through a Task. You have to build the entire list inside the method, finish the work, and then hand the completed collection over.

### Difference between Types

| Feature          | Task<IEnumerable<T>> |  IAsyncEnumerable<T>  |
|---------------|------------|---------------|----------------------|
|     Delivery Style     |    All-or-nothing (Batch).    | One-by-one (Stream). |
|     Memory usage    |    High: Must store the full list in memory before returning.    |       Low: Only stores the current item in memory.     |
|     Latency    |    High: Consumer waits for the last item to be ready.   |       Low: Consumer starts when the first item is ready.         |
| Async Flow |    Only the start/end of the collection is async   |       Every single iteration can be independently async.       |

```c#
Signature:
public interface IAsyncEnumerable<out T>
{
    IAsyncEnumerator<T> GetAsyncEnumerator(CancellationToken cancellationToken = default);
}

`IAsyncEnumerator<T>` has:

public interface IAsyncEnumerator<out T> : IAsyncDisposable
{
    ValueTask<bool> MoveNextAsync();
    T Current { get; }
}

```

### For Example
Your server calls the database to fetch 1000000 records, your normal optimization technique is to batch the data which is totally fine but then if you decide to batch in 10 pages that means you would be fetching 100000 records per page, you have to buffer 100000 records into memory (worst case you are fetching the entire row) before you start processing that. With IAsyncEnumerable<T>, you process items as they become available. 

// Task<IEnumerable<T>>
```c#
public async Task<IEnumerable<int>> GetDataAsync()
{
    var results = new List<int>(); 
    for (int i = 0; i < 100; i++)
    {
        await Task.Delay(100); // Wait...
        results.Add(i);        // Add to list...
    }
    return results; // Only now does the caller see ANYTHING.
}
```

// IAsyncEnumerable<T>
```c#
public async IAsyncEnumerable<int> GetNumbersAsync()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(500); // Simulating work like a DB call or API request
        yield return i; 
    }
}

//Consuming the Stream
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine(number); // This prints one by one every half-second
}
```

Cancellation Support
One "gotcha" is handling cancellation. You should always pass a CancellationToken to ensure your stream stops when the user or system requests it.

### When to use it?

- Reading rows from a database (Entity Framework Core)

- Streaming large files from disk

- Calling a paginated API


Test to make is going to be memory allocation