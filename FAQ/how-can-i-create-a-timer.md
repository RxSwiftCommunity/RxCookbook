# How can I create a timer?

I'd like to make an Observable that emits values periodically.

## Answer

Use `Observable<Int>.interval(_:scheduler:)`. It takes 2 parameters: `period: RxTimeInterval` and `scheduler: SchedulerType`. Pass the time interval (in seconds) to the first parameter, and pass the scheduler instance which the timer runs on.

This Observerable emits the total number of times the timer has fired, starting from `0`. Since the Observable can only emit integers, `interval(_:scheduler:)` method is only available on `Observable<Int>`.

The Observable created with `interval()` method emits values on every tick. For example, the code below emits interger on every 3 seconds.

```swift
Observable<Int>.interval(3, scheduler: MainScheduler.instance)
  .subscribe(onNext: { value in
    print(value)
  })
```

**Output**

```
[00:00:03.00] 0
[00:00:06.00] 1
[00:00:09.00] 2
[00:00:12.00] 3
```

### Creating a timer with an initial value

If you'd like to create a timer that emits an initial value immediately, you might consider using `startWith(_:)`.

```swift
Observable<Int>.interval(1.5, scheduler: MainScheduler.instance)
  .startWith(100)
  .subscribe(onNext: { value in
    print(value)
  })
```

**Output**

The first line will be printed just after the code is executed, and other lines will be printed on every 1.5 seconds.

```
[00:00:00.00] 100
[00:00:01.50] 0
[00:00:03.00] 1
[00:00:04.50] 2
[00:00:06.00] 3
```

### Limiting the number of time the timer fires

Sometimes we have to make the timer to execute specific number of times. `take(_:)` could be the best solution. Pass the number of values we'd like to take. The Observable will emit `complete` event just after emitting the last value.

```swift
Observable<Int>.interval(1, scheduler: MainScheduler.instance)
  .take(3)
  .subscribe { value in
    print(value)
  }
```

**Output**

```
[00:00:01.00] next(0)
[00:00:02.00] next(1)
[00:00:03.00] next(2)
[00:00:03.00] completed
```
