# Index
- [Index](#index)
- [Reactive Programming](#reactive-programming)
- [io.reactivex.rxjava3](#ioreactivexrxjava3)
  - [Overview](#overview)
    - [Observables](#observables)
    - [Simple background computation](#simple-background-computation)
    - [Some terminology](#some-terminology)
      - [Upstream, downstream](#upstream-downstream)
      - [Backpressure](#backpressure)
  - [Observable](#observable)
    - [Create source of Observable](#create-source-of-observable)
    - [Subscribe Observable](#subscribe-observable)
  - [Completable](#completable)
    - [Introduction](#introduction)

# Reactive Programming
- Asynchronous programming

# io.reactivex.rxjava3
- github: https://github.com/ReactiveX/RxJava
- It’s an API for asynchronous programming with observable streams.
- In this note, most basic concepts is illustrated in `Observable`.

## Overview
- Observable
  - It emits data and pushes it down the stream.
  - Single, Maybe, Flowable and Completable are another types of Observable.
- Observer
  - It listens (by subscription) for items emitted by the observable and consume it.

### Observables

Observables | Emit item | onSubscribe | onStart | onNext | onComplete | onSuccess | onError | Interface | Default observer
----------- | --------- | ----------- | ------- | ------ | ---------- | --------- | ------- | --------- | --------
`Observable<T>` | 0..N items, no backpressure | onSubscribe | | onNext | onComplete | | onError | `ObservableSource` | `Observer`
`Flowable<T>` | 0..N items, support backpressure | | onStart | onNext | onComplete | |  onError | `Publisher` | `Subscriber`
`Single<T>` | 1 item / Error | onSubscribe | | | | onSuccess |  onError | `SingleSource` | `SingleObserver`
`Maybe<T>` | 0..1 item / Error | onSubscribe | | | onComplete | onSuccess | onError | `MaybeSource` | `MaybeObserver`
`Completable` | (No items) Completion / Error | onSubscribe | | | onComplete | | onError | `CompletableSource` | `CompletableObserver`

*Note that - `onError` and `onComplete` are mutually exclusive.*

### Simple background computation

One of the common use cases for RxJava is to run some computation, network request on a background thread and show the results (or error) on the UI thread:
```java
Observable.fromArray(1,2,3,4,5)  // #1
    .subscribeOn(Schedulers.io())  // #2
    .observeOn(Schedulers.io())  // #2
    .map(integer -> {  // #2
        for(int i = 0; i < 100000000; i++){integer++;} //  imitate expensive computation
        return integer;
    })
    .subscribe(  // #3
        integer -> {  // #4
            System.out.println(integer + " Time now: " + Instant.now() + ", Thread: " + Thread.currentThread().getName());
        }
    );

System.out.println("Hi,       time now: " + Instant.now() + ", Thread: " + Thread.currentThread().getName());

Thread.sleep(2000); // <--- wait for the flow to finish

// Result:
// Hi,       time now: 2023-09-21T09:58:12.397Z, Thread: main
// 100000001 Time now: 2023-09-21T09:58:12.883Z, Thread: RxCachedThreadScheduler-2
// 100000002 Time now: 2023-09-21T09:58:13.283Z, Thread: RxCachedThreadScheduler-2
// 100000003 Time now: 2023-09-21T09:58:13.579Z, Thread: RxCachedThreadScheduler-2
// 100000004 Time now: 2023-09-21T09:58:13.975Z, Thread: RxCachedThreadScheduler-2
// 100000005 Time now: 2023-09-21T09:58:14.213Z, Thread: RxCachedThreadScheduler-2
```
There are four key parts:
1. An observable source
2. A number of chained operators which manipulate the stream values and its behavior
3. The subscription
4. Events callbacks

*Note that - the subscription happens immediately after calling `subscribe`, while event callbacks can happen anytime after that.*

Typically, you can move computations or blocking IO to some other thread via `subscribeOn`. Once the data is ready, you can make sure they get processed on the foreground or GUI thread via `observeOn`.

In RxJava the default `Schedulers` run on daemon threads, which means once the Java main thread exits, they all get stopped and background computations may never happen. Sleeping for some time in this example situations lets you see the output of the flow on the console with time to spare.

### Some terminology
#### Upstream, downstream
```java
source
    .operator1()
    .operator2()
    .operator3()
    .subscribe(observer);
```
Here, if we imagine ourselves on `operator2`, looking upwards /  towards the source is called the upstream. Looking downwards / towards the subscriber/consumer is called the downstream.

Note that in RxJava, the observables are immutable; each of the operation returns **a new observable** with added behaviour. To illustrate, the above code can be rewritten as follows:

```java
source = observable.create();

observable1 = source.operator1();

observable2 = observable1.operator2();

observable3 = observable2.operator3();

observable3.subscribe(observer);
```
Therefore, the order of `subscribeOn` and `observeOn` will affect the non-blocking behaviour.

#### Backpressure
When the dataflow runs through asynchronous steps, each step may perform different things with different speed. To avoid overwhelming such steps, which usually would manifest itself as increased memory usage due to temporary buffering or the need for skipping/dropping data, so-called backpressure is applied, which is a form of flow control where the steps can express how many items are they ready to process. **This allows constraining the memory usage of the dataflows in situations where there is generally no way for a step to know how many items the upstream will send to it.**

In RxJava, the dedicated `Flowable` class is designated to support backpressure and `Observable` is dedicated to the non-backpressured operations (short sequences, GUI interactions, etc.). The other types, `Single`, `Maybe` and `Completable` don't support backpressure nor should they; there is always room to store one item temporarily.

## Observable

```java
public abstract class Observable<T>
extends Object
implements ObservableSource<T>
```
- ref: http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Observable.html
- Default observer: `Observer`
- Protocol: `onSubscribe onNext* (onError | onComplete)?`

### Create source of Observable

There are many methods provided by the RxJava library for `Observable` creation:
- `create(@NonNull ObservableOnSubscribe<T> source)`

### Subscribe Observable

```java
Observable.fromArray(1,2,3,4,5)
        .map(new Function<Integer, Integer>()
        {
            @Override
            public Integer apply(Integer integer) throws Throwable {
                return integer * 10;
            }
        })
        .subscribe(new Observer<Integer>()
        {
            @Override
            public void onSubscribe(@NonNull Disposable d) {}

            @Override
            public void onNext(@NonNull Integer integer) {
                System.out.println(integer);
            }

            @Override
            public void onError(@NonNull Throwable e) {}

            @Override
            public void onComplete() {}
        });

// Simplified by lambda
Observable.fromArray(1,2,3,4,5)
        .map(integer -> integer * 10)
        .subscribe(  // onNext
            integer -> System.out.println(integer)
        );

Observable.fromArray(1,2,3,4,5)
        .map(integer -> integer * 10)
        .doOnNext(
            integer -> System.out.println(integer)
        )
        .subscribe();
```
`subscribe` is a terminal operator, which means it triggers the whole flow and you can't add any other operator after that.

`doOn...` is an intermediate operator which means you can use it almost anywhere in the chain even multiple times. Unlike `subscribe`, it doesn't trigger the execution. It comes in handy when you are working with longer and more complex reactive streams. For example when you want to log at multiple stages of the pipeline.

- Terminal operator (return `Disposable` / `void`)
  - `Disposable subscribe()` - Subscribes to the current `Observable` and ignores `onNext` and `onComplete` emissions. If the `Observable` emits an error, it is wrapped into an `OnErrorNotImplementedException` and routed to the `RxJavaPlugins.onError(Throwable)` handler. The follows are similar if they do not handle error emission.
  - `Disposable subscribe(@NonNull Consumer<? super T> onNext)`
  - `Disposable subscribe(@NonNull Consumer<? super T> onNext, @NonNull Consumer<? super Throwable> onError)`
  - `Disposable subscribe(@NonNull Consumer<? super T> onNext, @NonNull Consumer<? super Throwable> onError, @NonNull Action onComplete)`
  - `void subscribe(@NonNull Observer<? super T> observer)` - Subscribes the given `Observer` to the current `Observable`, handle all signal emissions.
- Intermediate operator (return `Observable`)
  - `Observable<T> doOnSubscribe(@NonNull Consumer<? super Disposable> onSubscribe)`
  - `Observable<T> doOnNext(@NonNull Consumer<? super T> onNext)`
  - `Observable<T> doOnError(@NonNull Consumer<? super Throwable> onError)`
  - `Observable<T> doOnComplete(@NonNull Action onComplete)`


## Completable

```java
public abstract class Completable
extends Object
implements CompletableSource
```
- ref: http://reactivex.io/RxJava/3.x/javadoc/io/reactivex/rxjava3/core/Completable.html

### Introduction

```java
Completable
        .fromAction(new Action()
        {
            @Override
            public void run() throws Throwable
            {
                System.out.println("running");
            }
        })
        .subscribe(new CompletableObserver()
        {
            @Override
            public void onSubscribe(@NonNull Disposable d)
            {

            }

            @Override
            public void onComplete()
            {
                System.out.println("finished");
            }

            @Override
            public void onError(@NonNull Throwable e)
            {

            }
        });
```