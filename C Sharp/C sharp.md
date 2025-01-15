# Index
- [Index](#index)
- [Overview](#overview)
- [Collections](#collections)
  - [HashSet](#hashset)
  - [List](#list)
  - [Tuple](#tuple)
- [Concurrent Collections](#concurrent-collections)
  - [ConcurrentDictionary](#concurrentdictionary)
  - [ConcurrentBag](#concurrentbag)
  - [ConcurrentQueue](#concurrentqueue)
- [Multithreading](#multithreading)
  - [Thread](#thread)
  - [Thread vs ThreadPool](#thread-vs-threadpool)
  - [Channels](#channels)
    - [Create a Channel](#create-a-channel)
    - [Design Producer](#design-producer)
    - [Design Consumer](#design-consumer)
- [Asychronous Programming](#asychronous-programming)
  - [Task](#task)
    - [Instantiate a Task](#instantiate-a-task)
      - [Task Constructor](#task-constructor)
      - [Task.Run](#taskrun)
    - [TaskCompletionSource](#taskcompletionsource)
  - [async/await](#asyncawait)
    - [Async function](#async-function)
    - [Async return types](#async-return-types)
    - [Best Practice](#best-practice)
    - [Deadlock](#deadlock)
    - [Thread](#thread-1)
    - [Async Main()](#async-main)
- [Iteration Statements](#iteration-statements)
  - [For Loop](#for-loop)
  - [Switch](#switch)
- [Access Modifiers](#access-modifiers)
- [Data Type](#data-type)
  - [Value Types and Reference Types](#value-types-and-reference-types)
  - [Array](#array)
  - [Class](#class)
    - [Fields](#fields)
    - [Properties](#properties)
  - [Enum](#enum)
  - [Struct](#struct)
  - [Integral Numeric Types](#integral-numeric-types)
    - [uint](#uint)
  - [Floating-point Numeric Types](#floating-point-numeric-types)
    - [Conversion from / to decimal](#conversion-from--to-decimal)
    - [decimal vs double](#decimal-vs-double)
  - [String](#string)
    - [Format](#format)
    - [Convert to Number](#convert-to-number)
- [Method Parameters](#method-parameters)
  - [Modifiers](#modifiers)
  - [Pass a value type by value](#pass-a-value-type-by-value)
  - [Pass a value type by reference](#pass-a-value-type-by-reference)
  - [Pass a reference type by value](#pass-a-reference-type-by-value)
  - [Pass a reference type by reference](#pass-a-reference-type-by-reference)
- [Keywords](#keywords)
    - [static](#static)
    - [const](#const)
    - [readonly](#readonly)
    - [required](#required)
    - [unsafe](#unsafe)
    - [delegate](#delegate)
    - [virtual](#virtual)
    - [using](#using)

# Overview

**Hello world**

```cs
using System;

public class Program
{
    public static void Main()
    {
        // This line prints "Hello, World" 
        Console.WriteLine("Hello, World");
    }
}
```
- The "Hello, World" program starts with a `using` directive that references the `System` namespace. Namespaces provide a hierarchical means (folders and files) of organizing C# programs and libraries. Because of the `using` directive, the program can use `Console.WriteLine` as shorthand for `System.Console.WriteLine`.

**Language Feature**

- C# is a part of Microsoft Visual Studio.
- C# is the most popular language for the .NET platform. 
- C# has good memory management and an automatic garbage collector.
- C# follows OOP concepts such as inheritance, abstraction, polymorphism, encapsulation.

**VS Java**

- C# uses CLR (Common Language Runtime), whereas Java uses JRE (Java Runtime Environment).
- C# is also object-oriented, functional, strongly typed, and **component-oriented**, while Java is only object-oriented.
- C# developers can use properties whereas Java needs get/set methods instead of properties.
- Java has built-in annotation processing which is absent in C#.
- C# uses less memory than Java, even though they both use a JIT compiler. Java doesn’t provide functions like Delete or Free, so directly, users have no control over garbage collection.

**Convention**

**Naming**

Kind | Rule
---- | ----
Field | UpperCamelCase
Property | UpperCamelCase
Method | UpperCamelCase
Local variable | lowerCamelCase
Parameter | lowerCamelCase

- Not to use underscores before any variable name or parameter name. ([stackoverflow](https://stackoverflow.com/questions/3136594/naming-convention-underscore-in-c-and-c-sharp-variables/3136618#3136618))

**Coding**

- If class `A` call the `Initialize` method of class `B`, then it is responsible to call the `Shutdown` method of class `B` also.

**Visual Studio**

- [Solutions and Projects](https://learn.microsoft.com/en-us/visualstudio/ide/solutions-and-projects-in-visual-studio?view=vs-2022)
- Type `///` to write summary.

# Collections

## HashSet

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1)
- A set is a collection that contains no duplicate elements, and whose elements are in no particular order.

```cs
set_name.Add(item)  // return bool, false if already exist
set_name.Remove(item)
```

## List

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)
- Allow duplicates.

```cs
list_name.Count  // Number of items in the list
list_name[list_name.Count - 1]  // Get the last item if not empty

list_name.FindAll(val => val == 1)  // Get all elements filtering by a predicate
list_name.Find(val => val == 1)  // Get the first element filtering by a predicate

list_name.RemoveAll(predicate)

list_name.Contains(item)  // return bool


list_name.GetRange(10, 15) // get 15 items from index 10
// With paging (offset and limit), e.g. get the third page, with each page 10 items
int offset = 2; // zero-based
int limit = 10;
if (list_name.Count > offset * limit)
{
    list_name.GetRange(offset * limit, Math.Min(limit, list_name.Count - offset * limit))
}


// List<string>
string.Join(",", list_name) // concatenate the strings in list with ,
```

## Tuple

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)

```cs
(double, int) t1 = (4.5, 3);
```

# Concurrent Collections
- Reads and writes of the following data types shall be atomic: bool, char, byte, sbyte, short, ushort, uint, int, float, and reference types. [ref](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables#atomicity-of-variable-references)
- Interlocked.Exchange

## ConcurrentDictionary

- [ConcurrentDictionary<TKey,TValue> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2)
- [KeyValuePair<TKey,TValue> Struct](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.keyvaluepair-2)

```cs
// Number of key/value pairs in the dictionary
cd_name.Count

// Instead of Count == 0
cd_name.IsEmpty

// For loop if you need both key and value
foreach (var pair in cd_name)
{
    var key = pair.Key;
    var value = pair.Value;
}

// For loop if you only need either key or value
foreach (var key in cd_name.Keys)
foreach (var value in cd_name.Values)

// Get a list of keys or value
cd_name.Keys.ToList()
cd_name.Values.ToList()

// Get a value by key
if (cd_name.TryGetValue(key, out var value) && value != null)

// Add a value, return false if the key already exists
if (cd_name.TryAdd(key, value))

// Add or update a key/value pair
cd_name.AddOrUpdate(key, newValue, (key, oldValue) => {return newValue})
```
- Note that the dictionary checks key existence by comparing `GetHashCode()`.
  - This allows the key to be a `Tuple`.
- Although all methods of `ConcurrentDictionary` are thread-safe, not all methods are atomic, specifically `GetOrAdd` and `AddOrUpdate`.
- Not recommend to filter by value, as it is much slower than that by key.

## ConcurrentBag

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1)

## ConcurrentQueue

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1)
- FIFO

```cs
IsEmpty
Clear()
ToArray()
cq.Enqueue()
TryDequeue()  // remove and return the first object
TryPeek()  // return without remove the first object
```

# Multithreading

## Thread
```cs
using System;
using System.Threading;

class Program
{
    static void Main(string[] args)
    {
        Thread t = new Thread(() => {
            // Do some work
        })
        t.Start();

        // Wait the worker thread to finish
        t.Join();
    }
}
```

```cs
Thread.CurrentThread.Name
```

## Thread vs ThreadPool

- `ThreadPool` is reusing threads that have already been created. Once your machine is sufficiently busy, the `ThreadPool` will queue up requests rather than immediately spawn more threads. But `Thread` is always creating new ones (an expensive process).
- `ThreadPool` threads are background threads that will stop when the main thread ends. Manually created `Thread` are foreground by default (will keep running after the main thread has ended), but can be set to background before calling `Start()` on them.
- `ThreadPool` sizes itself based on the current workload and available hardware and is optimised for a large number of relatively short-lived operations. Creating a new `Thread` yourself would be more appropriate if the job were going to be relatively long running.

## Channels

- [Document](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)

This library is available in the `System.Threading.Channels` NuGet package. However, if you're using .NET Core 3.0 or later, the package is included as part of the framework.

Channels are an implementation of the producer/consumer conceptual programming model. In this programming model, producers asynchronously produce data, and consumers asynchronously consume that data. In other words, this model hands off data from one party to another.

To use channels:

1. Create a channel.
2. Design producer.
3. Design consumer.

### Create a Channel

To create an *unbounded channel*, call one of the `Channel.CreateUnbounded()` overloads:

```cs
var channel = Channel.CreateUnbounded<T>();
```

When you create an unbounded channel, by default, the channel can be used by any number of readers and writers concurrently. Alternatively, you can specify nondefault behavior when creating an unbounded channel by providing an `UnboundedChannelOptions` instance. The channel's capacity is unbounded and all writes are performed synchronously.

To create a *bounded channel*, call one of the `Channel.CreateBounded()` overloads:

```cs
var channel = Channel.CreateBounded<T>(7);
```

The preceding code creates a channel that has a maximum capacity of 7 items. When you create a bounded channel, the channel is bound to a maximum capacity. When the bound is reached, the default behavior is that the channel asynchronously blocks the producer until space becomes available. You can configure this behavior by specifying an option when you create the channel.

### Design Producer

- [Methods reference](https://learn.microsoft.com/en-us/dotnet/api/system.threading.channels.channelwriter-1)

Use `TryWrite()`:

```cs
var channel = Channel.CreateUnbounded<string>();

if (channel.Writer.TryWrite("message ..."))
{
    ...
}
```

`TryWrite()` will try to write the specified item to the channel.

- When used with an *unbounded channel*, this always returns `true` unless the channel is marked / is being marked as completed.
- When used with an *bounded channel*, this might return `false` if the channel reaches the maximum bound of writer.

Use `WriteAsync()`:

```cs
var channel = Channel.CreateUnbounded<string>();

await channel.Writer.WriteAsync("message ...");
```

`WriteAsync()` will ensure the specified item is written to the channel. When it is waiting to write, the method is blocked asynchronously.

### Design Consumer

```cs

```

# Asychronous Programming

- [Asynchronous programming with async and await - Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)

## Task

- [Task Class - Microsoft](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task)
- [Task Class Supplementation - Microsoft](https://learn.microsoft.com/en-us/dotnet/fundamentals/runtime-libraries/system-threading-tasks-task)

The `Task` class represents a single operation that does not return a value and that usually executes asynchronously. For operations that return values, you use the `Task<T>` class.

`Task` objects are one of the central components of the *task-based asynchronous pattern* first introduced in the .NET Framework 4.

### Instantiate a Task

#### Task Constructor

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

Action<object> action = (object obj) =>
                        {
                            Console.WriteLine("Task={0}, obj={1}, Thread={2}",
                            Task.CurrentId, obj, Thread.CurrentThread.ManagedThreadId);
                        };


// Create a task but do not start it.
Task t1 = new Task(action, "alpha");

// Launch t1 asynchronously on another thread
t1.Start();
Console.WriteLine("t1 has been launched. (Main Thread={0})", Thread.CurrentThread.ManagedThreadId);

// Wait for the task to finish.
t1.Wait();


// Construct an unstarted task
Task t2 = new Task(action, "beta");

// Run it synchronously on the main thread
t2.RunSynchronously();
Console.WriteLine("t2 has been executed. (Main Thread={0})", Thread.CurrentThread.ManagedThreadId);

// Although the task was run synchronously, it is a good practice
// to wait for it in the event exceptions were thrown by the task.
t2.Wait();
```

```cs
// The example displays output like the following:
//       t1 has been launched. (Main Thread=1)
//       Task=1, obj=alpha, Thread=4
//       t2 has been executed. (Main Thread=1)
//       Task=2, obj=beta, Thread=1
```

If a `Task` is instantiated by calling a `Task` class constructor, 

- it is started asynchronously on one or more thread pool threads by calling its `Start()` method, or 
- it is started synchronously on the main application thread by calling its `RunSynchronously()` method.

#### Task.Run

- [Task.Run Method](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.run)
- [Task.Run vs TaskFactory.StartNew](https://devblogs.microsoft.com/pfxteam/task-run-vs-task-factory-startnew/)

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

// Construct a started task using Task.Run.
String taskData = "delta";
Task t3 = Task.Run(() =>
{
    Console.WriteLine("Task={0}, obj={1}, Thread={2}",
            Task.CurrentId, taskData, Thread.CurrentThread.ManagedThreadId);
});

// Wait for the task to finish.
t3.Wait();

// The example displays output like the following:
//       Task=3, obj=delta, Thread=3
```

If a `Task` is instantiated by calling the `Task.Run(...)` method, 

- it is started after instantiation.
- it executes asynchronously typically on one or more thread pool threads. (called `.NET ThreadPool Worker`)

To allow the work to be cancelled, use a cancellation token. Note that the exception is catched when calling `await`.
- If your purpose is cancelling the task when it has not yet started, use `Task.Run(Action, CancellationToken)`. It throws `TaskCanceledException`.
- If your purpose is cancelling the task when it has started, just use `Task.Run(Action)`. Inside the action, use `token.IsCancellationRequested` to check and call `token.ThrowIfCancellationRequested()`. It throws `OperationCanceledException`.

```cs
var c = new CancellationTokenSource();
Task t = Task.Run(() =>
{
    while (true)
    {
        if (c.Token.IsCancellationRequested)
            c.Token.ThrowIfCancellationRequested();
        logger.Info("hi");
        Task.Delay(1000).Wait();
    }
});
Thread.Sleep(5000);
c.Cancel();
try
{
    await t;
}
catch (Exception ex) 
{
    logger.Error(ex);
}

// Result:
//     hi
//     hi
//     hi
//     hi
//     OperationCanceledException
```

### TaskCompletionSource

This allow us to manually control when and how a task completes. For example, you want to set a timeout for a method.

Note that `source.Task` is a `Task`.

```cs
private ConcurrentDictionary<string, TaskCompletionSource<bool>> 
    loginTaskConDict = new ConcurrentDictionary<string, TaskCompletionSource<bool>>();

public async Task<bool> Login(string login, string password)
{
    var source = new TaskCompletionSource<bool>();

    if (loginTaskConDict.TryAdd(login, source))
    {
        // ... send a login request

        Task timeout = Task.Delay(30000);
        Task receiveLoginResponse = source.Task;

        // waiting any task to be completed first
        // if the first completed task is source.Task, then wait for its result
        if (receiveLoginResponse == await Task.WhenAny(receiveLoginResponse, timeout))
        {
            return await receiveLoginResponse;  // waiting the task to be completed
        }
    }
    return false;
}

// on receive a login response
public void OnLoginResposne(string login)
{
    if (loginTaskConDict.TryRemove(login, out TaskCompletionSource<bool> source))
        source.SetResult(true);  // set the task to be completed with result
}
```

## async/await

- [async](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async)
- [await](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await)

The goal of `async/await` syntax - enable code that reads like a sequence of statements, but executes in a much more complicated order based on external resource allocation and when tasks are complete.

The `async` keyword turns a method into an async method, which allows you to use the `await` keyword in its body.

When the `await` keyword is applied, it suspends the calling method and yields control back to its caller until the awaited task is complete.

- For I/O-bound code, you **await an operation** that returns a `Task` or `Task<T>` inside of an async method. You *should not* use the Task Parallel Library.
- For CPU-bound code, you **await an operation** that is started on a background thread with the `Task.Run` method.

### Async function

- [Task asynchronous programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model)

![](../image/async-program-flow.png)

1. A calling method calls and awaits the `GetUrlContentLengthAsync` async method.
2. `GetUrlContentLengthAsync` creates an `HttpClient` instance and calls the `GetStringAsync` asynchronous method to download the contents of a website as a string.
3. Something happens in `GetStringAsync` that suspends its progress. It needs some time to return a `string` result. To avoid blocking resources, `GetStringAsync` yields control to its caller, `GetUrlContentLengthAsync`, by returning a task `Task<string>`. 
   
   `GetUrlContentLengthAsync` assigns the task to the `getStringTask` variable. The task represents the ongoing process for the call to GetStringAsync, with a commitment to produce an actual string value when the work is complete. (i.e. the work is started)
4. Because `getStringTask` hasn't been awaited yet, `GetUrlContentLengthAsync` can continue with other work that doesn't depend on the final result from `GetStringAsync`. That work is represented by a call to the synchronous method `DoIndependentWork`.
5. `DoIndependentWork` is a synchronous method that does its work and returns to its caller.
6. `GetUrlContentLengthAsync` has run out of work that it can do without a result from `getStringTask`. It must wait for the result of the task.
   
   Therefore, `GetUrlContentLengthAsync` uses an await operator to suspend its progress and to yield control to the method that called `GetUrlContentLengthAsync`. `GetUrlContentLengthAsync` returns a `Task<int>` to the caller. The task represents a promise to produce an integer result that's the length of the downloaded string.

   Inside the calling method the processing pattern continues. The caller might do other work that doesn't depend on the result from `GetUrlContentLengthAsync` before awaiting that result, or the caller might await immediately. The calling method is waiting for `GetUrlContentLengthAsync`, and `GetUrlContentLengthAsync` is waiting for `GetStringAsync`.

   Note that if `GetUrlContentLengthAsync` doesn't have to wait for the final result, the expense of suspending and then returning to `GetUrlContentLengthAsync` would be wasted.
7. `GetStringAsync` completes and produces a string result. The string result isn't returned by the call to `GetStringAsync` in the way that you might expect. (Remember that the method already returned a task in step 3.) **Instead, the string result is stored in the task that represents the completion of the method**, `getStringTask`. The await operator retrieves the result from `getStringTask`. The assignment statement assigns the retrieved result to `contents`.
8. When `GetUrlContentLengthAsync` has the string result, the method can calculate the length of the string. Then the work of `GetUrlContentLengthAsync` is also complete, and the waiting event handler can resume.

### Async return types

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)

Async methods can have the following return types:
- `Task`, for an async method that performs an operation but returns no value.
- `Task<TResult>`, for an async method that returns a value.
- `void`, for an event handler. It can't be awaited, so it must let the main thread know when it finished, e.g. using `TaskCompletionSource`.

### Best Practice

With async programming, there are some details to keep in mind that can prevent unexpected behavior.

- `async` methods need to have an `await` keyword in their body or they will never yield.
- If `await` is not used in the body of an `async` method, the C# compiler generates a warning, but the code compiles and runs as if it were a normal method. This is incredibly inefficient, as the state machine generated by the C# compiler for the async method is not accomplishing anything.
- Add "Async" as the suffix of every async method name you write.
- `async void` should only be used for event handlers.
- Async All the Way - Write code that awaits `Task` in a non-blocking manner. This is also easier to handle exceptions.

The following is a cheat sheet of async replacements for synchronous operations:

Use this... | Instead of this... | When
----------- | ------------------ | ----
await task | task.Wait() or task.Result | Retrieving the result of a background task
await Task.WhenAny(task1, task2, ...) | Task.WaitAny(task[])	| Waiting for any task to complete
await Task.WhenAll(task1, task2, ...) | Task.WaitAll(task[])	| Waiting for all tasks to complete
await Task.Delay(1000) | Thread.Sleep(1000) | Waiting for a period of time

Every `Task` will store a list of exceptions. When you await a `Task`, the first exception is re-thrown, so you can catch the specific exception type (such as `InvalidOperationException`). However, when you synchronously block on a `Task` using `Task.Wait` or `Task.Result`, all of the exceptions are wrapped in an `AggregateException` and thrown.

For loop:
- Use a `List` to store all tasks and use `WhenAll` to wait for all task finish.

### Deadlock

This example is extracted from the [Best practice](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming) for GUI and network thread.
```cs
public static class DeadlockDemo
{
  private static async Task DelayAsync()
  {
    await Task.Delay(1000);
  }
  // This method causes a deadlock when called in a GUI or ASP.NET context.
  public static void Test()
  {
    // Start the delay.
    var delayTask = DelayAsync();
    // Wait for the delay to complete.
    delayTask.Wait();
  }
}
```

The root cause of this deadlock is due to the way `await` handles contexts. By default, when an incomplete `Task` is awaited, the current “context” is captured and used to resume the method when the `Task` completes. This “context” is the *current SynchronizationContext* unless it’s null, in which case it’s the current `TaskScheduler`. GUI and ASP.NET applications have a *SynchronizationContext* that permits **only one chunk of code to run at a time**. When the `await` completes, it attempts to execute the remainder of the async method within the captured context. But that context already has a thread in it, which is (synchronously) waiting for the async method to complete. They’re each waiting for the other, causing a deadlock.

Note that console applications don’t cause this deadlock. They have a **thread pool** *SynchronizationContext* instead of a one-chunk-at-a-time *SynchronizationContext*, so when the `await` completes, **it schedules the remainder of the async method on a thread pool thread**. The method is able to complete, which completes its returned task, and there’s no deadlock.\

To avoid deadlock, add `ConfigureAwait(false)` (may not work?). This time, when the await completes, it attempts to execute the remainder of the async method within the thread pool context. The method is able to complete, which completes its returned task, and there’s no deadlock.

### Thread

The `async` and `await` keywords **don't cause additional threads to be created**. Async methods don't require multithreading because an async method doesn't run on its own thread. The method runs on the *current SynchronizationContext* and uses time on the thread only when the method is active.

You can use `Task.Run` to move CPU-bound work to a background thread, but a background thread doesn't help with a process that's just waiting for results to become available.

### Async Main()

Before C# 7.1 version, if you really want to use async method in the `Main()` method:

```cs
class Program
{
    static void Main(string[] args)
    {
        Task.Run(async () =>
        {
            // ...
        }).Wait();
    }
}
```

After C# 7.1 version, you can turn the `Main()` method into an async method, so that the whole program can be asynchronous.

```cs
class Program
{
    static async Task Main(string[] args)
    {
        // ...
    }
}
```

However, don't do this in .NET console application. If the Main method were async, it could return before it completed, causing the program to end. Use the following instead.

```cs
class Program
{
    static void Main()
    {
        MainAsync().Wait();
    }

    static async Task MainAsync()
    {
        // ...
    }
}
```

# Iteration Statements

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements)

## For Loop

```cs
for (int i = 0; i < 3; i++)
{
    Console.Write(i);
}

var fibNumbers = new List<int> { 0, 1, 1, 2, 3, 5, 8, 13 };
foreach (int element in fibNumbers)
{
    Console.Write($"{element} ");
}

await foreach (var item in GenerateSequenceAsync())
{
    Console.WriteLine(item);
}
```

## Switch

```cs
private string GetPeriodTableName(EnPeriodType periodType)
{
    return periodType switch
    {
        EnPeriodType.OneMin => Constant.OneMinDbTable,
        EnPeriodType.FiveMin => Constant.FiveMinDbTable,
        EnPeriodType.FifteenMin => Constant.FifteenMinDbTable,
        EnPeriodType.ThirtyMin => Constant.ThirtyMinDbTable,
        EnPeriodType.OneHour => Constant.OneHourDbTable,
        EnPeriodType.FourHour => Constant.FourHourDbTable,
        EnPeriodType.OneDay => Constant.OneDayDbTable,
        EnPeriodType.OneWeek => Constant.OneWeekDbTable,
        EnPeriodType.OneMonth => Constant.OneMonthDbTable,
        _ => "",
    };
}
```

# Access Modifiers

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)

# Data Type

## Value Types and Reference Types

- [Document](https://learn.microsoft.com/en-us/dotnet/visual-basic/programming-guide/language-features/data-types/value-types-and-reference-types)
- [Reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)
- [Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
- [Default values](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/default-values)

Value Type | Default Value
---------- | -------------
Numeric | 0
bool | false
enum | (E)0
struct | all fields set to default

Reference Type | Default Value
-------------- | -------------
object | null
string | null
array | null

## Array

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays)

```cs
// Declare a single-dimensional array of 5 integers.
int[] array1 = new int[5];

// Declare and set array element values.
int[] array2 = [1, 2, 3, 4, 5, 6];

array_name.Length;
```

## Class

### Fields

```cs
public class Genre
{
    public string Name;
}
```

### Properties

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties)
- [Auto-implemented properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties)

```cs
public class Genre
{
    public string Name { get; set; }
}
```

The `{ get; set; }` is called auto-implemented property, which is a shorthand for the following code. Similar code will be generated by the compiler.

```cs
public class Genre
{
    private string _name;  // This is the backing field
    public string Name
    {
        get
        {
            return this.name;
        }
        set
        {
            this.name = value;
        }
    }
}
```

It is different from just declaring a public field as you can control which accessor is allowed. You can just declare `get;` or `set;`, just like a private field with getter or setter method.

## Enum

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum)

```cs
using System;

// Common usage
public enum EnOrderType
{
    OP_BUY = 0,
    OP_SELL = 1,
    OP_BUY_LIMIT = 2,
    OP_SELL_LIMIT = 3,
    ...
}

// Can cast to any integral numeric date type
(int)EnOrderType.OP_BUY        // 0

// Sometimes need to add toString to pass as string
EnOrderType.OP_BUY.toString()  // "OP_BUY"

// Cast integral numeric data type to enum
(EnOrderType)0  // "OP_BUY"

// Check if a integral value is valid for a enum type
Enum.IsDefined(typeof(EnOrderType), -1)  // false
Enum.IsDefined(typeof(EnOrderType), 0)  // true

// For loop a enum
foreach (EnOrderType type in Enum.GetValues(typeof(EnOrderType)))

// Total number of items in a enum
Enum.GetValues(typeof(EnOrderType)).Length
```

## Struct

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
- All fields, if not initialized in the constructor, will be set to default value.

## Integral Numeric Types

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/integral-numeric-types)

C# type/keyword	| Range	| Size | .NET type
--------------- | ----- | ---- | ---------
sbyte   | -128 to 127 | Signed 8-bit integer   | System.SByte
byte    | 0 to 255    | Unsigned 8-bit integer | System.Byte
short   | -32,768 to 32,767	| Signed 16-bit integer    | System.Int16
ushort  | 0 to 65,535		| Unsigned 16-bit integer  | System.UInt16
int	    | -2,147,483,648 to 2,147,483,647 | Signed 32-bit integer   | System.Int32
uint    | 0 to 4,294,967,295			  | Unsigned 32-bit integer	| System.UInt32
long    | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | Signed 64-bit integer | System.Int64
ulong   | 0 to 18,446,744,073,709,551,615						  | Unsigned 64-bit integer | System.UInt64
nint    | Depends on platform (computed at runtime)	| Signed 32-bit or 64-bit integer | System.IntPtr
nuint   | Depends on platform (computed at runtime)	| Unsigned 32-bit or 64-bit integer | System.UIntPtr

### uint
```cs
// uint to int
uint n = 3;

int i = (int)n;  // unchecked, will be negative if n > Int32 max value
int i = Convert.ToInt32(n);  // checked, will throw exception if n > Int32 max value

// int to uint
int n = 10;

uint i = (uint)n;  // unchecked, will > Int32 max value if n is negative
uint i = Convert.ToUInt32(n);  // checked, will throw exception if n is negative
```

```cs
long largestValue = long.MaxValue;
```

## Floating-point Numeric Types

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types)

C# type/keyword | Approximate range | Precision | Size
--------------- | ----------------- | --------- | ----
float   | ±1.5 x 10e−45 to ±3.4 x 10e38    | ~6-9 digits   | 4 btyes
double  | ±5.0 × 10e−324 to ±1.7 × 10e308  | ~15-17 digits | 8 bytes
decimal | ±1.0 x 10e-28 to ±7.9228 x 10e28 | 28-29 digits  | 16 bytes

```cs
float x = 1f / 3f;
double y = 1d / 3d;
decimal z = 1m / 3m;

Console.WriteLine(x);  // 0.33333334
Console.WriteLine(y);  // 0.3333333333333333
Console.WriteLine(z);  // 0.3333333333333333333333333333
```

### Conversion from / to decimal
```cs
Convert.ToDecimal()  // * to decimal

Decimal.ToUInt64()  // decimal to ulong
```

### decimal vs double
- https://exceptionnotfound.net/decimal-vs-double-and-other-tips-about-number-types-in-net/

The `decimal` type is appropriate when the required degree of precision is determined by the number of digits to the right of the decimal point. Such numbers are commonly used in financial applications, for currency amounts (for example, $1.00), interest rates (for example, 2.625%), and so forth. 

Even numbers that are precise to only one decimal digit are handled more accurately by the `decimal` type: 0.1, for example, can be exactly represented by a `decimal` instance, while there's no `double` or `float` instance that exactly represents 0.1. Because of this difference in numeric types, unexpected rounding errors can occur in arithmetic calculations when you use `double` or `float` for decimal data.

Notes:
- `double` uses base-2 math, while `decimal` uses base-10 math.
- `double` is faster than `decimal`.
- `double` uses less storage than `decimal`.

## String

- https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/

### Format

Quoted string literals (Normal string):
```cs
string columns = "Column 1\tColumn 2\tColumn 3";
//Output: Column 1        Column 2        Column 3

string rows = "Row 1\r\nRow 2\r\nRow 3";
/* Output:
    Row 1
    Row 2
    Row 3
*/

string title = "\"The \u00C6olean Harp\", by Samuel Taylor Coleridge";
//Output: "The Æolean Harp", by Samuel Taylor Coleridge
```

Verbatim string literals:

```cs
string filePath = @"C:\Users\scoleridge\Documents\";
//Output: C:\Users\scoleridge\Documents\

string text = @"My pensive SARA ! thy soft cheek reclined
    Thus on mine arm, most soothing sweet it is
    To sit beside our Cot,...";
/* Output:
My pensive SARA ! thy soft cheek reclined
    Thus on mine arm, most soothing sweet it is
    To sit beside our Cot,...
*/

string quote = @"Her name was ""Sara.""";
//Output: Her name was "Sara."
```

- It is more convenient for multi-line strings, strings that contain backslash characters, or embedded double quotes.

String interpolation

```cs
var jh = (firstName: "Jupiter", lastName: "Hammon", born: 1711, published: 1761);
Console.WriteLine($"{jh.firstName} {jh.lastName} was an African American poet born in {jh.born}.");
Console.WriteLine($"He was first published in {jh.published} at the age of {jh.published - jh.born}.");
Console.WriteLine($"He'd be over {Math.Round((2018d - jh.born) / 100d) * 100d} years old today.");

// Output:
// Jupiter Hammon was an African American poet born in 1711.
// He was first published in 1761 at the age of 50.
// He'd be over 300 years old today.
```

- Interpolated strings are identified by the `$` special character and include interpolated expressions in braces.
- String interpolation achieves the same results as the `String.Format` method, but improves ease of use and inline clarity.

### Convert to Number

```cs
try
{
    int m = Int32.Parse("abc");
}
catch (FormatException e)
{
    Console.WriteLine(e.Message);
}
// Output: Input string was not in a correct format.

if (Int32.TryParse("abc", out int numValue))
{
    Console.WriteLine(numValue);
}
else
{
    Console.WriteLine($"Int32.TryParse could not parse '{inputString}' to an int.");
}
// Output: Int32.TryParse could not parse 'abc' to an int.
```

# Method Parameters

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)

In C#, arguments can be passed to method parameters either by **value** or by **reference**.
- Pass by value means passing a copy of the variable to the method.
- Pass by reference means passing access to the variable to the method.

## Modifiers
Parameters declared for a method without `in`, `ref` or `out`, are passed to the called method by value. The `ref`, `in`, and `out` modifiers differ in assignment rules:
- The argument for a `ref` parameter must be definitely assigned. The called method may reassign that parameter.
- The argument for an `in` parameter must be definitely assigned. The called method can't reassign that parameter.
- The argument for an `out` parameter needn't be definitely assigned. The called method must assign the parameter.


You can't use the in, ref, and out keywords for the following kinds of methods:
- Async methods, which you define by using the `async` modifier.
- Iterator methods, which include a yield return or `yield break` statement.

## Pass a value type by value
```cs
int n = 5;
System.Console.WriteLine("The value before calling the method: {0}", n);

SquareIt(n);  // Passing the variable by value.
System.Console.WriteLine("The value after calling the method: {0}", n);

// Keep the console window open in debug mode.
System.Console.WriteLine("Press any key to exit.");
System.Console.ReadKey();

static void SquareIt(int x)
// The parameter x is passed by value.
// Changes to x will not affect the original value of x.
{
    x *= x;
    System.Console.WriteLine("The value inside the method: {0}", x);
}
/* Output:
    The value before calling the method: 5
    The value inside the method: 25
    The value after calling the method: 5
*/
```

## Pass a value type by reference
```cs
int n = 5;
System.Console.WriteLine("The value before calling the method: {0}", n);

SquareIt(ref n);  // Passing the variable by reference.
System.Console.WriteLine("The value after calling the method: {0}", n);

// Keep the console window open in debug mode.
System.Console.WriteLine("Press any key to exit.");
System.Console.ReadKey();

static void SquareIt(ref int x)           // <-- different
// The parameter x is passed by reference.
// Changes to x will affect the original value of x.
{
    x *= x;
    System.Console.WriteLine("The value inside the method: {0}", x);
}
/* Output:
    The value before calling the method: 5
    The value inside the method: 25
    The value after calling the method: 25   <-- different
*/
```

## Pass a reference type by value
```cs
int[] arr = { 1, 4, 5 };
System.Console.WriteLine("Inside Main, before calling the method, the first element is: {0}", arr[0]);

Change(arr);
System.Console.WriteLine("Inside Main, after calling the method, the first element is: {0}", arr[0]);

static void Change(int[] pArray)
{
    pArray[0] = 888;  // This change affects the original element.
    pArray = new int[5] { -3, -1, -2, -3, -4 };   // This change is local.
    System.Console.WriteLine("Inside the method, the first element is: {0}", pArray[0]);
}
/* Output:
    Inside Main, before calling the method, the first element is: 1
    Inside the method, the first element is: -3
    Inside Main, after calling the method, the first element is: 888
*/
```

## Pass a reference type by reference
```cs
int[] arr = { 1, 4, 5 };
System.Console.WriteLine("Inside Main, before calling the method, the first element is: {0}", arr[0]);

Change(ref arr);
System.Console.WriteLine("Inside Main, after calling the method, the first element is: {0}", arr[0]);

static void Change(ref int[] pArray)
{
    // Both of the following changes will affect the original variables:
    pArray[0] = 888;
    pArray = new int[5] { -3, -1, -2, -3, -4 };
    System.Console.WriteLine("Inside the method, the first element is: {0}", pArray[0]);
}
/* Output:
    Inside Main, before calling the method, the first element is: 1
    Inside the method, the first element is: -3
    Inside Main, after calling the method, the first element is: -3
*/
```


# Keywords

### static

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members)

A static class is basically the same as a non-static class, except that it: 
- Cannot be instantiated. In other words, you cannot use the `new` operator to create a variable of the static class type.
- Contains only static members.
- Is sealed, which means cannot be inherited. It cannot inherit from any class or interface except `Object`.

### const

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const)

A `const` field can only be initialized at the declaration of the field. *It is a compile-time constant* which is stored in metadata.

They are not instance variable, which is similar to `static`.

### readonly

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly)
- [Const vs Readonly](https://www.bytehide.com/blog/const-vs-readonly-in-c-explanation-in-3-minutes)

A `readonly` field can be initialized at the declaration or in a constructor in the same class. *It is a run-time constant* which is stored in the heap memory. A `readonly` field can be assigned and reassigned multiple times within the field declaration and constructor.

```cs
class Age
{
    private readonly int _year;
    Age(int year)
    {
        _year = year;
    }
    void ChangeYear()
    {
        //_year = 1967; // Compile error if uncommented.
    }
}
```

This rule has different implications for value types and reference types:

- Because value types directly contain their data, a field that is a `readonly` value type is immutable.
- Because reference types contain a reference to their data, a field that is a `readonly` reference type must always refer to the same object. That object isn't immutable. The `readonly` modifier prevents the field from being replaced by a different instance of the reference type. However, the modifier doesn't prevent the instance data of the field from being modified through the read-only field.

### required

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required)

The `required` modifier indicates that the field or property it's applied to must be initialized by an object initializer (not constructor). Any expression that initializes a new instance of the type must initialize all *required* members. The `required` modifier is available beginning with C# 11.

```cs
public class OrderResponse
{
    public          ulong Login   { get; set; }
    public required uint  RtnCode { get; set; }
}

{
    // compile error, does not initialize the property "RtnCode"
    OrderResponse response = new OrderResponse()
    {
        Login = 123,
    }
}
```

### unsafe

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/unsafe)

The `unsafe` keyword denotes an unsafe context, which is required for any operation involving pointers.

In short: can use pointer in C#.

### delegate

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)

In short: method pointer in C#.

This is a instance variable which is **a type of a method**. You can assign a custom method as a variable, with the same parameter type and return type.

```cs
public class Worker
{
    // It becomes a type
    public delegate int PerformCalculation(int x, int y);

    public Worker(PerformCalculation method)
    {
        method(1,2);
    }
}

public class Program
{
    public static void Main()
    {
        // Pass the method as a parameter
        new Worker(Addition);
    }

    public int Addition(int x, int y)
    {
        ...
    }
}
```

### virtual

- [Document](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual)

The `virtual` keyword is used to modify a method, property, indexer, or event declaration and allow for it to be overridden in a derived class.

By default, methods are non-virtual. You cannot override a non-virtual method.

Virtual methods have an implementation and provide the derived classes with the option of overriding it. Abstract methods do not provide an implementation and force the derived classes to override the method.

### using

`using` statement

The `using` statement is used to define a scope for an object that implements the `IDisposable` interface. The `IDisposable` interface defines a method called `Dispose()` that is used to release any unmanaged resources used by the object, such as file handles or database connections. When you use the `using` statement with an object that implements `IDisposable`, the object is automatically disposed of when the end of the `using` block is reached. Here's an example:
```cs
void Method()
{
    using (MySqlConnection dbConn = new MySqlConnection(dbConfig))
    {
        // do some work

    } --> // dbConn.Dispose() is called here
}
```
In this example, the `MySqlConnection` object is automatically disposed of when the end of the `using` block is reached.

`using` declaration

```cs
void Method()
{
    using MySqlConnection dbConn = new MySqlConnection(dbConfig)
    
    // do some work
  
} --> // dbConn.Dispose() is called here
```

- Note that it requires C# 8.0 or above.

`using` directive

The `using` directive is used to import namespaces into your C# code. A namespace is a way to group related types together, and the `using` directive allows you to reference types within a namespace without having to fully qualify their names.

