---
title: Asynchronous programming with async and await
draft: false
tags:
  - software-development
  - asynchronous-programming
  - csharp
  - dotnet
---
## Introduction

Successful modern applications require asynchronous code. Asynchronous programming allows tasks to run concurrently without blocking the main thread. One thread can handle instructions asynchronously, meaning a task can be started, and the system manages them, turning its attention to tasks that are ready for further processing. For a parallel algorithm multiple threads are necessary [[#1]]. If used correctly, asynchronous programming can improve the efficiency of code execution by allowing tasks to be performed concurrently, without unnecessarily blocking computational resources.

Multithreading allows you to increase the responsiveness of your application and, if your application runs on a multiprocessor or multi-core system, increase its throughput. So, it’s important not to block threads because those threads could be serving other requests [[#1]] [[#2]].

> **Process**
> A process is an executing program. An operating system uses processes to separate the applications that are being executed. A process can contain multiple threads [[#2]].
> 
> **Thread**
> A thread is the basic unit to which an operating system allocates processor time. A thread can execute any part of the program code, including parts currently being executed by another thread [[#2]].

### Asynchronous programming using C\#

> **Check out this project where I implemented the practical use of Task in C#:** [lucashmalcantara/tap-csharp: Asynchronous programming with async and await using C#.](https://github.com/lucashmalcantara/tap-csharp)
> 
> The implementation demonstrates how Task behaves in C# under different scenarios.

Below is a list of the main features related to asynchronous programming using C#:

- The `await` keyword provides a non-blocking way to start a task, then continue execution when that task completes.
- In C#, a `Task` or `Task<TResult>` represents an asynchronous operation, that runs concurrently in a thread.
- A `Task` starts execution either at the moment it's called or when it's assigned to a variable.
  
```csharp
var eggsTask = FryEggsAsync(2); // The task starts running here.
...
var eggs = await eggsTask; // Wait for the result to be ready.
Console.WriteLine("eggs are ready");
```

- When a task that runs asynchronously throws an exception, that Task is faulted. The Task object holds the exception thrown in the [Task.Exception](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.exception#system-threading-tasks-task-exception) property. Faulted tasks throw an exception when they're awaited.
- Faulted tasks store the exception into [Task.Exception](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.exception#system-threading-tasks-task-exception) property. This property is a [System.AggregateException](https://learn.microsoft.com/en-us/dotnet/api/system.aggregateexception) because more than one exception may be thrown during asynchronous work.
- When you `await` a `Task`, the `AggregateException` is automatically unpacked, and only the first exception is thrown.
- Avoid Using [Task<>.Result ](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1.result) on a Task. Accessing `Result` on a faulted task immediately rethrows the exception and skips over any subsequent code. Instead, `await` the task to handle the exception properly in an `async` context.
  
```csharp
var reportTask = GenerateReportAsync();

// BAD
await reportTask;
Console.WriteLine(reportTask.Result);

// GOOD
var result = await reportTask;
Console.WriteLine(result);
```

## Analyzing codes

### Preparing breakfast

Are the following two C# codes equivalent?

First:

```csharp
Coffee cup = PourCoffee();
Console.WriteLine("Coffee is ready");

Task<Egg> eggsTask = FryEggsAsync(2);
Task<Bacon> baconTask = FryBaconAsync(3);
Task<Toast> toastTask = ToastBreadAsync(2);

Toast toast = await toastTask;
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("Toast is ready");
Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

Egg eggs = await eggsTask;
Console.WriteLine("Eggs are ready");
Bacon bacon = await baconTask;
Console.WriteLine("Bacon is ready");

Console.WriteLine("Breakfast is ready!");
```


Second:

```csharp
Coffee cup = PourCoffee();
Console.WriteLine("Coffee is ready");

Toast toast = await ToastBreadAsync(2);
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("Toast is ready");
Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

Egg eggs = await FryEggsAsync(2);
Console.WriteLine("Eggs are ready");
Bacon bacon = await FryBaconAsync(3);
Console.WriteLine("Bacon is ready");

Console.WriteLine("Breakfast is ready!");
```

The two codes are **not equivalent**. The key difference lies in the order and concurrency of the tasks.

#### Analysis of the first code

-   The coffee is poured immediately.
-   Three tasks are started concurrently:
    -   Frying eggs (`FryEggsAsync`),
    -   Frying bacon (`FryBaconAsync`),
    -   Toasting bread (`ToastBreadAsync`).
-   The program then waits for the toast task to complete, applies butter and jam, and moves on without waiting for the eggs or bacon to finish.
-   After the toast is ready, orange juice is poured.
-   The program **awaits the tasks for eggs and bacon sequentially**, meaning it waits for them to finish at the end.

#### Analysis of the second code

-   The coffee is poured immediately.
-   The program **awaits the `ToastBreadAsync` task before continuing**, meaning it waits for the toast to complete before starting the tasks for eggs or bacon.
-   After the toast is ready, butter and jam are applied, and orange juice is poured.
-   The program **awaits `FryEggsAsync` and `FryBaconAsync` sequentially**, so these tasks only start after the toast is done.

#### Key Differences

1.  **Concurrency:**
    -   **First Code:** Tasks for eggs, bacon, and toast run concurrently. This can result in better performance since the tasks are asynchronous and do not block the main thread.
    -   **Second Code:** Tasks for eggs and bacon do not start until the toast is ready, resulting in **no concurrency**.
2.  **Execution Order:**
    -   **First Code:** The toast is prepared while the eggs and bacon are frying in parallel.
    -   **Second Code:** The toast is prepared first, then the eggs are fried, and finally, the bacon is fried.

#### Implications

-   **First Code** is more efficient since it uses concurrency to overlap the preparation of toast, eggs, and bacon.
-   **Second Code** is simpler and easier to follow but could take longer because it doesn't utilize the concurrency potential of asynchronous tasks.

#### Which is Better?

-   If reducing breakfast preparation time is a priority, the **first code** is better because it uses the `Task` API effectively to run tasks concurrently.
-   If simplicity and clarity are more important, the **second code** might be preferred.

## References

### 1

Microsoft. “Asynchronous programming with async and await.” microsoft.com. Accessed: Jan. 26, 2025. [Online.] Available:
https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/
### 2

Microsoft. “Threads and threading.” microsoft.com. Accessed: Jan. 27, 2025. [Online.] Available: https://learn.microsoft.com/en-us/dotnet/standard/threading/threads-and-threading
## Related content

- [The Task Asynchronous Programming (TAP) model with async and await" - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model)