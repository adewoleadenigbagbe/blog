---
layout: post
title: 'Task vs ValueTask'
date: 2026-05-26 12:00:00
categories:
  - csharp
---

### Introduction
Task and ValueTask are fundamentals tools in .NET used by developers when they work on asynchronous programming but still get confused. Task was introduced to .NET 4.0 while ValueTask was introduced to .NET Core 2.1. They both allow applications to be responsive why executing a long running operations and also scalability.I am going to explain the difference between Task and ValueTask, the usecases, strength and limitations.

### Scenario
In Async world, we do a lot of I/O operation such as Database, External api calls, interacting with the file system even though concurrently we are actually managing threads and not having a solely dedicated thread for asynchronous calls, there still a little drawback with Task because Task object get allocated in the heap and much more a drawback if we find ourselves requesting for data that we already have. For example, you are calling an endpoint many times to fetch data but you want to check if you have the data in memory before you make the http call,regardless of whether the data is found in memory it get allocated on the Heap.Imagine having to make a hundreds/thousands of calls means you would have an all the object stored on the Heap.

This is where ValueTask shines, ValueTask is a **struct** (value type) introduced to solve the allocation overhead of Task. It is a perfomance tool that addresses this, it can be synchronous as well as asynchronous, synchronous in the sense that we check to see if we have the data, if we do it return the data, if we dont then make asychronous call, in that way we are saving allocation Task object on the Heap 

#### Example
```c#
public async ValueTask<int> GetDataAsync() 
{
    if (isCached) 
    {
       return 42; // No heap allocation!
    }
    
    return await FetchFromDbAsync(); // Only allocates if it actually goes to the DB
}
```
### Benchmark Test
Here we try to benchmark the difference between using Task and ValueTask, we have two benchmark operations

SimulateValueTaskOperation() iterates through a number of times with postIds, calls GetDataValueTask(id) method to check the cache to see if data can be found, if not then call the endpoint to fetch the post,it return a ValueTask<Post>

![Value task operation]({{ '/images/image-1.png' | relative_url }})


SimulateTaskOperation() does the same but calls GetDataTask(id) and return a Task<Post>

![Task operation]({{ '/images/image-2.png' | relative_url }})

After Benchmark, the result below for the SimulateValueTaskOperation the average mean is little faster than SimulateTaskOperation, but we are more interested in the allocated space in memory for the objects, SimulateValueTaskOperation averages 13.69kb while SimulateTaskOperation averages 20.52kb. The difference might not be too much because we the iterations was in a couple of hundreds in this example, imagine we did for iteration for thousands of hundred or millions.

![Benchmark Result]({{ '/images/image-3.png' | relative_url }})

### Limitations
While ValueTask is a performance tool, it is not a replacement of Task, there are some limitation to it

#### Store it and later await later
❌ **Bad**
```c#
var v1 =  ValueTaskMethod()
await v1
```

#### Await a ValueTask multiple times
❌ **Bad**
```c#
var v1 =  ValueTaskMethod()
var result1 = await v1
var result2 = await v1
```

#### Call a GetAwaiter and GetResult
❌ **Bad**
```c#
var v1 =  ValueTaskMethod()
var result = v1.GetAwaiter().GetResult()
```

#### Running ValueTask Concurrently
❌ **Bad**
```c#
var v1 =  ValueTaskMethod()
Task.Run(async () => await v1)
Task.Run(async () => await v1)
```

#### Mental Model
ValueTask is not a replacement for Task, it should be used when you think about memory optimazation as well as when you have operation that can be synchronous or asynchronous.


#### Final
You can get code samples for this post here [Task Vs ValueTask](https://github.com/adewoleadenigbagbe/Blog-Code-Samples/tree/main/TaskVsValueTask)


#### Thanks for reading!

If you found this post helpful, feel free to drop a comment and share it with others who might benefit from it.
