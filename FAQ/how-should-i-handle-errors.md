# How should I handle errors?

I'd like to know how should I handle errors in general.

## Answer

First I think we have to define what *Error* is. In Rx world, whenever error happens in a sequence, this sequence is terminated:
> Okay, this shouldn't happen here, ever. I'm gonna crash (terminate).

Now when we look at the common "Errors", like a network error, do we really want to completely terminate the sequence and doesn't accept anything anymore? Depends.

Imagine you have a `UITableView` with cats, populated by the API. To gather info about them, you make a request, map the data to objects and then bind to `UITableView`. What happens if we got an error there? If we don't protect ourselves, the sequence terminates, thus the `UITableView` data source terminates and we cannot update this view with new data again.

### How do we protect ourselves?
We have multiple choices.

#### 1. Catching errors.
RxSwift has two methods that are responsible for catching the error. Specifically:

```swift
.catchErrorJustReturn(_:) // 1
.catchError(_:) // 2
```

First one should return an element, while the second one should return an `Observable` sequence, that can guide the original sequence how to recover. In fact, these operators below are equal:

```swift
.catchErrorJustReturn([])
.catchError { _ in Observable.just([]) }
```

**_Warning!_** Whenever you use catching in your sequence, remember that original sequence is basically gone. It won't emit any items again. You can either emit one item after the error `.catchErrorJustReturn([])` or emit how many you want, with the new sequence using `catchError(_:)`.

**_Example:_**

#### 2. Retrying.

#### 3. Driver

#### 4. Result
