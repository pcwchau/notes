# Index

- [Index](#index)
- [Concurrency](#concurrency)
  - [History of concurrency](#history-of-concurrency)
  - [Create a new thread](#create-a-new-thread)
  - [Interface](#interface)
    - [Callable](#callable)
    - [Runnable](#runnable)
  - [Executors](#executors)
    - [Create a ExecutorService](#create-a-executorservice)
- [Thread Safety](#thread-safety)
  - [Atomicity](#atomicity)
    - [Atomic Variable](#atomic-variable)
    - [Intrinsic Lock](#intrinsic-lock)
      - [Reentrancy](#reentrancy)
  - [Memory Visibility](#memory-visibility)
    - [Intrinsic Lock](#intrinsic-lock-1)
    - [Volatile](#volatile)
- [Asynchronous Programming](#asynchronous-programming)
  - [Future](#future)
  - [Process Results of Future](#process-results-of-future)
  - [FutureTask](#futuretask)
    - [ExecutorService submit()](#executorservice-submit)
  - [CompletableFuture](#completablefuture)
    - [CompletableFuture Constructor](#completablefuture-constructor)
    - [CompletableFuture.supplyAsync()](#completablefuturesupplyasync)
    - [After Completion](#after-completion)

# Concurrency

- The entry thread when a Java application starts up is *main* thread.

## History of concurrency

In the acient past, computers did not have operating systems. They could only execute a single program from beginning to end. 

Operating systems evolved to allow more than one program to run at once, running individual programs in *processes*: isolated, independently executing programs to which the operating system allocates resources such as memory, file handles, and security credentials.

Later, *threads* were developed to allow multiple streams of program control flow to coexist within a *process*. They share process-wide resources such as *heap* and file handles, but each thread has its own program counter, *stack*, and local variables.

> *Heap* - The shared memory area among threads where all the objects live. Any thread that has a reference to the object in heap can read or modify the object.

> *Stack* - The private memory area allocated to each of the running thread. For example, two threads calling a common utility method simultaneously will execute the method within its own stack where local variables and method arguments of one thread is not visible to the other thread.

```java
public static void main(String[] args) {
  Calculator calc = new Calculator();

  Thread user1 = new Thread(() -> calc.add(1, 2));
  Thread user2 = new Thread(() -> calc.add(3, 4));

  user1.start();
  user2.start();
}

static class Calculator {
  public int add(int a, int b) {
    int c = a + b;
    return c;
  }
}
```

In this example, `calc` is stored in the *heap*, while local variable `a`, `b`, and `c` are stored in the *stack*.

## Create a new thread

There are two ways to create a new thread of execution.

The first one is to declare a class to be a subclass of `Thread`. This subclass should override the `run()` method of class `Thread`. An instance of the subclass can then be allocated and started.

```java
class PrimeThread extends Thread {
  public void run() {
    // ...
  }
}

PrimeThread p = new PrimeThread();
p.start();
```

The other way is to declare a class that implements the `Runnable` interface. That class then implements the `run()` method. An instance of the class can then be allocated, passed as an argument when creating a `Thread`, and started.

```java
class PrimeRun implements Runnable {
  public void run() {
    // ...
  }
}

PrimeRun p = new PrimeRun();
new Thread(p).start();
```

Using `Runnable` is more common because the class can still extend other parent class.

## Interface

### Callable

- [Interface Callable - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Callable.html)

```java
package java.util.concurrent;

@FunctionalInterface
public interface Callable<V> {

    /**
     *Computes a result, or throws an exception if unable to do so.
     */
    V call() throws Exception;
}
```

A task that returns a result and **may throw an exception**. Implementors define a single method with no arguments called `call`.

The `Callable` interface is similar to `Runnable`, in that both are designed for classes whose instances are potentially executed by another thread. A `Runnable`, however, does not return a result and cannot throw a checked exception.

The `Executors` class contains utility methods to convert from other common forms to `Callable` classes.

### Runnable

- [Interface Runnable - Java 8](https://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)

```java
@FunctionalInterface
public interface Runnable {

    /**
     * Runs this operation.
     */
    void run();
}
```

Methods | Usage
--------|------
run(): void | When it is called by `start()` of a thread, run the task in separately executing thread.

## Executors

- [Class Executors - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html)
- [Interface ExecutorService - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)

### Create a ExecutorService

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

private ExecutorService executor1 = Executors.newSingleThreadExecutor();
private ExecutorService executor2 = Executors.newFixedThreadPool(10);
private ExecutorService executor3 = Executors.newCachedThreadPool();
```

`Executors` Static Method | Description | ExecutorService
------ | ----------- | ---------------
`newSingleThreadExecutor()` | Creates an Executor that uses a single worker thread operating off an unbounded queue. | `ThreadPoolExecutor`
`newFixedThreadPool(int n)` | Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue. | `ThreadPoolExecutor`
`newCachedThreadPool()` | Creates a thread pool that creates new threads as needed, but will reuse previously constructed threads when they are available. | `ThreadPoolExecutor`

# Thread Safety

Writing thread-safe code is, at its core, about managing access to *shared*, *mutable* variable.

> *Shared* - A variable is *shared* if it could be accessed by multiple threads.

> *Mutable* - A variable is *mutable* if its value could change during its lifetime.

Whenever more than one thread accesses a given variable, and one of them might write to it, they all must coordinate their access to it using synchronization in the following aspects:

- Atomicity - Prevent one thread from modifying the variable of an object when another is using it; and
- Memory visibility - Ensure that when a thread modifies the variable of an object, other threads can actually see the changes that were made.

The primary mechanism for synchronization in Java is the `synchronized` keyword, which provides exclusive locking, but the term synchronization also includes the use of `volatile` variables, explicit locks, and atomic variables.

## Atomicity

```java
import java.util.concurrent.Executors;

public class ThreadTestDepositWithoutSync {
    public static void main(String[] args) {
        var account = new Account();
        var executor = Executors.newCachedThreadPool();

        for (int i = 0; i < 100; i++) {
            executor.execute(() -> account.deposit(1));
        }

        executor.shutdown();

        while (!executor.isTerminated()) {}

        System.out.println("Current account balance: " + account.getBalance());
    }

    private static class Account {
        private int balance = 0;

        public int getBalance() {
            return balance;
        }

        public void deposit(int amount) {
            balance += amount; // read-modify-write operation
        }
    }
}
```

This is an example of *read-modify-write* operation, which shows what can happen if hundred threads try to increment a variable simultaneously without synchronization. 

The correct result is 100, but sometimes the result is smaller than 100. With some unlucky timing where more than one thread reads the same value of `balance`, and each add one to it, some increments got lost along the way, causing an inaccurate result. We can say that the operation, `balance += amount`, is not *atomic*, which means that it does not execute as a single, indivisible operation.

This problem is called a race condition, which occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime; in other words, when getting the right answer relies on lucky timing.

Another type of race condition is called *check-then-act*. See the following example.

```java
public class LazyInitRace {
    private ExpensiveObject instance = null;

    @NotThreadSafe
    public ExpensiveObject getInstance() {
        if (instance == null) // check-then-act operation
            instance = new ExpensiveObject();
        return instance;
    }
}
```

Say that threads A and B execute `getInstance()` at the same time. A sees that `instance` is `null`, and instantiates a new `ExpensiveObject`. B also checks if `instance` is `null`. Whether `instance` is `null` at this point depends unpredictably on timing, including the vagaries of scheduling and how long A takes to instantiate the `ExpensiveObject` and set the `instance` field. If `instance` is `null` when B examines it, the two callers to `getInstance()` may receive two different results, even though `getInstance()` is always supposed to return the same instance.

To avoid race conditions, we need to ensure the read-modify-write and check-then-act operations are *atomic*.

> *Atomic* - Operations A and B are *atomic* with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same variable.

There must be a way to prevent other threads from using a variable while we are in the middle of modifying it, so we can ensure that other threads can observe or modify the variable only before we start or after we finish, but not in the middle.

### Atomic Variable

- [Package java.util.concurrent.atomic - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/package-summary.html)

The `java.util.concurrent.atomic` package contains atomic variable classes for effecting atomic state transitions on numbers and object references. In the following example, by replacing the `int` variable with an `AtomicInteger`, we ensure that all operations that access the `balance` variable are atomic.

```java
import java.util.concurrent.atomic.AtomicInteger;

private static class Account {
    private AtomicInteger balance = new AtomicInteger(0);

    public int getBalance() {
        return balance.get();
    }

    public void deposit(int amount) {
        balance.addAndGet(amount);
    }
}
```

**Warning: more than one atomic variable**

If you have multiple atomic variables that are dependent, even though they are individually thread-safe, the class might still have race conditions. It is because we cannot update all related atomic variables simultaneously.

### Intrinsic Lock

Java provides a built-in locking mechanism for enforcing atomicity: the `synchronized` block. A `synchronized` block has two parts:

- A reference to an object that will serve as the lock.
- A block of code to be guarded by that lock.

JVM assigns a lock to each class and object for purposes of synchronization; these built-in locks are called intrinsic locks or monitor locks.

The lock is automatically acquired by the executing thread before entering a `synchronized` block and automatically released when control exits the `synchronized` block, whether by the normal control path or by throwing an exception out of the block.

There are three kinds of `synchronized` block:

- A `synchronized` statement, whose lock is the specific object.
- A `synchronized` instance method, whose lock is the object on which the method is being invoked.
- A `synchronized` static method, whose lock is the class on which the method is being invoked.

Intrinsic locks in Java act as mutexes (or mutual exclusion locks), which means that at most one thread may own the lock. When thread A attempts to acquire a lock held by thread B, A must wait, or block, until B releases it. If B never releases the lock, A waits forever.

Since only one thread at a time can execute a block of code guarded by a given lock, no thread executing a `synchronized` block can observe another thread to be in the middle of a `synchronized` block guarded by the same lock.

In the following example, we make all methods that access the `balance` variable `synchronized`. **It is a common mistake to assume that synchronization needs to be used only when writing to shared variables; this is simply not true. (Read [Memory Visibility](#memory-visibility) for more details)**

```java
private static class Account {
    private int balance = 0;

    // Reading a shared variable also needs synchronization
    public synchronized int getBalance() {
        return balance;
    }

    public synchronized void deposit(int amount) {
        balance += amount;
    }
}
```

In the following example, if a `synchronized` method contains some long-running operations that do not affect shared variable, we can narrow the scope of the `synchronized` block by using `synchronized` statements.

```java
private static class Account {
    private int balance = 0;

    public synchronized int getBalance() {
        return balance;
    }

    public void deposit(int amount) {
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) { }

        synchronized (this) {
            balance += amount;
        }
    }
}
```

You should be careful not to make the scope of the `synchronized` block too small; you would not want to divide an operation that should be atomic into more than one `synchronized` block. Also, acquiring and releasing a lock has some overhead, so it is undesirable to break down `synchronized` blocks too far.

#### Reentrancy

When a thread requests a lock that is already held by another thread, the requesting thread blocks. But because intrinsic locks are reentrant, if a thread tries to acquire a lock that it already holds, the request succeeds. Reentrancy means that locks are acquired on a per-thread rather than per-invocation basis.

Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the `synchronized` block, the count is decremented. When the count reaches zero, the lock is released.

Without reentrant locks, the very natural-looking code in the following example, in which a subclass overrides a `synchronized` method and then calls the superclass method, would deadlock. Because the `doSomething` methods in `Widget` and `LoggingWidget` are both `synchronized`, each tries to acquire the lock on the object `myWidget` before proceeding. But if intrinsic locks were not reentrant, the call to `super.doSomething` would never be able to acquire the lock because it would be considered already held, and the thread would permanently stall waiting for a lock it can never acquire. Reentrancy saves us from deadlock in situations like this.

```java
public class Widget {
    public synchronized void doSomething() {
        ...
    }
}

public class LoggingWidget extends Widget {
    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}

// Execution
LoggingWidget myWidget = new LoggingWidget();
myWidget.doSomething();
```

## Memory Visibility



### Intrinsic Lock

### Volatile


**Experience 1**
```java
// Suppose a socket is created and shared between Thread A and Thread B.

// Thread A
// If the socket is closed, then it will become null.
socket.closeHandler(() -> {
  socket = null;
})

// Thread B
// If the socket is not null, it will keep sending message.
while (socket != null) {
  socket.write("hi");
}
```
- Since the two statements `socket = null` and `socket != null` are executed in separate threads, there is no guarantee that Thread A's change to the `socket` will be visible to Thread B.
- Suppose `socket = null` is executed in Thread A. Then, shortly afterwards, `socket != null` is checked in Thread B. It is possible that the `socket != null` is `true` in Thread B at that moment, and throw `NullPointerException` inside the while loop.
- In this case, the *happens-before* relationship is simply a guarantee that after executing `socket = null` in Thread A, `socket != null` in Thread B must be `false`.
- Note: You can use a **boolean variable (flag)** to control the behaviour after the socket closed, instead of changing it to `null` to avoid always checking `null`, although it also does not guarantee the *happens-before* relationship.


- Language spec: https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.4

By using `volatile`, we can communicate with runtime and processor to not reorder any instruction involving the volatile variable. Also, processors understand that they should immediately flush any updates to these variables before next read. **(atomic only if write and subsequently read, no mutual exclusion)**

**Example - Usage**
```java
public class TestVolatile {
    private static boolean stop = false;

    public static void main(String[] args) {
        // Thread-A
        new Thread("Thread A") {() -> {
                while (!stop) { }
                System.out.println(Thread.currentThread() + " stopped");
            }
        }.start();

        // Thread-main
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(Thread.currentThread() + " after 1 seconds");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        stop = true;
    }
}

// Result:
// Thread[main,5,main] after 1 seconds

// Process finished with exit code 0
```
Thread A is keep looping, because it cannot read the updated value of `stop` from Thread Main. To avoid this, add `volatile` for the `stop` variable.
```java
private static volatile boolean stop = false;

// Result:
// Thread[main,5,main] after 1 seconds
// Thread[Thread A,5,main] stopped

// Process finished with exit code 0
```

**Example - Non-atomic**
```java
public class VolatileTest01 {
    volatile int i;

    public void addI(){
        i++;
    }

    public static void main(String[] args) throws InterruptedException {
        final  VolatileTest01 test01 = new VolatileTest01();
        for (int n = 0; n < 1000; n++) {
            new Thread(() -> {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                test01.addI();
            }).start();
        }
        Thread.sleep(10000);
        System.out.println(test01.i);
    }
}
// Result:
// 981
```
Normally, we would expect the result is 1000. However, `volatile` cannot guarantee the atomicity of the variable `i` if the operation is `i++`. For example, when Thread A is doing `i++` for `i == 0`, it is possible that Thread B is also doing `i++` for `i == 0` instead of for `i == 1`. 

Note that `i++` has three steps: 
- Read the value of `i`
- Add 1 to the read value
- Write the value to `i`

`volatile` can only guarantee that if Thread A has finished `i++`, then Thread B will get the updated value of `i`. It does not guarantee mutual exclusion.

# Asynchronous Programming

- [Asynchronous Programming in Java - Baeldung](https://www.baeldung.com/java-asynchronous-programming)

## Future

- [Guide To Future - Baeldung](https://www.baeldung.com/java-future)

The `Future` interface was added in Java 5 to serve as a result of an asynchronous computation. This result will eventually appear in the `Future` after the processing is complete.

```java
package java.util.concurrent;

public interface Future<V>
```

Long running methods are good candidates for asynchronous processing and the `Future` interface because we can execute other processes while we’re waiting for the task encapsulated in the `Future` to complete. For example:

- computational intensive processes (mathematical and scientific calculations)
- manipulating large data structures (big data)
- remote method calls (downloading files, HTML scrapping, web services)

Note that `Future` is an interface, so we need to use its implementing classes:

- `CompletableFuture`
- `CountedCompleter`
- `ForkJoinTask`
- `FutureTask` (simple one)
- `RecursiveAction`
- `RecursiveTask`
- `SwingWorker`

## Process Results of Future

- [Interface Future - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)
- [Interface Future - Java 21](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html)

The result can only be retrieved using the `get()` method when the computation has completed, blocking if necessary until it is ready. Cancellation is performed by the `cancel()` method. Once a computation has completed, the computation cannot be cancelled.

Here, we get a returned `Future` from the above example:

```java
Future<Integer> future = new SquareCalculator().calculate(10);

while(!future.isDone())
{
    System.out.println("Calculating...");
    Thread.sleep(300);
}

Integer result = future.get();
System.out.println("Result: " + result);
```

```java
// The example displays output like the following:
// Calculating...
// Calculating...
// Calculating...
// Calculating...
// Result: 100
```

It’s worth mentioning that `get()` has an overloaded version that takes a timeout and a `TimeUnit` as arguments:

```java
Integer result = future.get(500, TimeUnit.MILLISECONDS);
```

After calling the `get()`, if the task doesn’t return a result before the specified timeout period, it will throw a `TimeoutException`.

## FutureTask

### ExecutorService submit()

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class SquareCalculator
{
    private ExecutorService executor = Executors.newCachedThreadPool();
    
    public FutureTask<Integer> calculate(Integer input)
    {
        return executor.submit(() -> 
        {
            Thread.sleep(1000);
            return input * input;
        });
    }
}
```

Here, we use the `Executors.newCachedThreadPool()` to create an `ExecutorService` object. Then, we have to call the `submit()` method of the `ExecutorService`, passing a `Callable` object as an argument. The `Callable` object is created using lambda expression.

Then, `submit()` will start the task in a new thread, depending on the type of the `ExecutorService`, and immediately return a `FutureTask` object implementing the `Future` interface.

## CompletableFuture

- [Class CompletableFuture - Java 8](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
- [Guide To CompletableFuture - Baeldung](https://www.baeldung.com/java-completablefuture)
- [Asynchronous API with CompletableFuture: Performance Tips and Tricks - Oracle](https://qconsf.com/sf2017/system/files/presentation-slides/cf.pdf)

Java 8 introduced the `CompletableFuture` class. Along with the `Future` interface, it also implemented the `CompletionStage` interface. This interface defines the contract for an asynchronous computation step that we can combine with other steps or handle possible error.

```java
package java.util.concurrent;

public class CompletableFuture<T>
extends Object
implements Future<T>, CompletionStage<T>
```

### CompletableFuture Constructor

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SquareCalculator
{
    private ExecutorService executor = Executors.newCachedThreadPool();
    
    public CompletableFuture<Integer> calculate(Integer input)
    {
        CompletableFuture<Integer> completableFuture = new CompletableFuture<>();

        executor.execute(() ->
        {
            try
            {
              Thread.sleep(1000);
            }
            catch (InterruptedException e)
            {
              throw new RuntimeException(e);
            }
            completableFuture.complete(input * input);
        });

        return completableFuture;
    }
}
```

Here, we use the `Executors.newCachedThreadPool()` to create an `ExecutorService` object. Also, we create a `CompletableFuture` object by its no-args constructor. Then, we have to call the `execute()` method of the `ExecutorService`, passing a `Runnable` object as an argument. The `Runnable` object is created using lambda expression.

Then, `execute()` will start the task in a new thread, depending on the type of the `ExecutorService`, and immediately return `void`.

Finally, we return the `CompletableFuture` object created by ourselves. To indicate that the task is done, result is provided to the `complete()` method.

### CompletableFuture.supplyAsync()

```java
import java.util.concurrent.CompletableFuture;

public class SquareCalculator
{
    public CompletableFuture<Integer> calculate(Integer input)
    {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> 
        {
            try
            {
              Thread.sleep(1000);
            }
            catch (InterruptedException e)
            {
              throw new RuntimeException(e);
            }
            return input * input;
        });

        return completableFuture;
    }
}
```

Here, we create a `CompletableFuture` object by its static `supplyAsync()` method, passing a `Supplier` object as an argument. The `supplier` object is created using lambda expression.

Then, `supplyAsync()` will start the task in a new thread, by default in the `ForkJoinPool.commonPool()`, and immediately return a `CompletableFuture` object implementing the `Future` interface.

Note that if you do not want to use the default executor, `supplyAsync()` allows passing a specific executor.

### After Completion

We can define the next task of a `CompletableFuture` after the task wrapped inside it has completed. The next task will also be executed asynchronously.

| Method       | Apply           | Accept           | Run              |
|--------------|-----------------|------------------|------------------|
| Single Input | `thenApply`     | `thenAccept`     | `thenRun`        |
| Binary OR    | `applyToEither` | `acceptEither`   | `runAfterEither` |
| Binary AND   | `thenCombine`   | `thenAcceptBoth` | `runAfterBoth`   |

Method Form | Example | Description
----------- | ------- | -----------
`method(...)` | `thenApply(...)` | Default execution by the same executor of this stage.
`methodAsync(...)` | `thenApplyAsync(...)` | `methodAsync(..., ForkJoinPool.commonPool())`.
`methodAsync(..., executor)` | `thenApplyAsync(..., executor)` | Runs method chain in the specific `executor`.


After the task wrapped in a `CompletableFuture` is done, 

If the next task needs to process the result, call the `thenApply()` method, passing a `Function` object as an argument:

```java
CompletableFuture<Integer> completableFuture = new SquareCalculator().calculate(10);

CompletableFuture<Integer> newCompletableFuture = completableFuture.thenApply(i -> i + 100);
```

If the next task needs to process the result, but do not need to return anything further, call the `thenAccept()` method, passing a `Consumer` object as an argument:

```java
CompletableFuture<Integer> completableFuture = new SquareCalculator().calculate(10);

CompletableFuture<Void> newCompletableFuture = completableFuture.thenAccept(i -> System.out.println(i));
```

If the next task neither needs the result nor wants to return some value further, call the `thenRun()` method, passing a `Runnable` object as an argument:

```java
CompletableFuture<Integer> completableFuture = new SquareCalculator().calculate(10);

CompletableFuture<Void> newCompletableFuture = completableFuture.thenRun(() -> System.out.println("Done"));
```