# How should I handle errors?

I'd like to know how should I handle errors in general.

## Answer

First I think we have to define what *Error* is. In Rx world, whenever error happens in a sequence, this sequence is terminated:
> Okay, this shouldn't happen here, ever. I'm gonna crash (terminate).

Now when we look at the common "Errors", like a network error, do we really want to completely terminate the sequence and don't accept anything anymore? Depends.

Imagine you have a `UITableView` with cats, populated by the API. To gather info about them, you make a request, map the data to objects and then bind to `UITableView`. What happens if we got an error there? If we don't protect ourselves, the sequence terminates, thus the `UITableView` data source terminates and we cannot update this view with new data again.

> ### But how do we protect ourselves?

We have multiple choices.

## 1. Catching errors.
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

**_Warnings:_**
👮 Whenever you use catching in your sequence, remember that original sequence is basically gone. It won't emit any items again. You can either emit one item after the error `.catchErrorJustReturn([])` or emit how many you want, with the new sequence using `catchError(_:)`.

**_Example:_**
```swift
// This is a button that refreshes your table view
let refreshTableViewObservable = refreshButton.rx.tap

refreshTableViewObservable
    .flatMapLatest { _ in
        return Api.rx_request()
            .mapArray(Cat.self)
            // Below we catch the error in flatMap, why?
            // Because if we extract it outside the flatMap,
            // it will end the whole original sequence instead
            // of the inner sequence that is just temporary.
            // This is really important.
            .catchErrorJustReturn([])
    }
    .bindTo(tableView.rx.items) { (tv, i, catName) in
        let cell = tv.dequeueReusableCell(withIdentifier: "catIndetifier", for: IndexPath(row: i, section: 0))
        cell.textLabel?.text = "\(catName)"

        return cell
    }
    .addDisposableTo(disposeBag)
```

## 2. Retrying.
Whenever error happens in a sequence you can also use `retry` operators. They basically go through entire sequence once error happens (based on a condition or not). We have 3 `retry` operators:

```swift
.retry()
```
This one retries the whole sequence whenever error happens. It has one advantage: you probably don't have to worry about errors (at least from this sequence).

**_Warnings:_**<br />
👮 This is really dangerous and you have to be careful when using it - one mistake and you can create infinity loop in your app. (Because it retries until there is a success)

```swift
.retry(_:)
```
This one is probably the most popular. It takes a parameter which indicates how many times **_in total_** the sequence will proceed in case of an error.

**_Warnings:_**<br />
👮 `retry(1)` does basically nothing. If you encounter an error and want it to retry once, then you must use `retry(2)`.<br />
👮 When you use this form of `retry(n)`, you must take into account that even after `n-1` retries error can happen. You might have to protect yourself e.g. using [catching](#1-catching-errors).

```swift
.retryWhen(_:)
```
This one is the most powerful one. You can retry the sequence based on anything you want. Let's say you want to retry you sequence whenever `URLError` happens. And only twice (so 3 requests in total max). This is all possible with `retryWhen(_:)`:

```swift
...
refreshTableViewObservable
    .debug()
    .flatMapLatest { _ in
        return Api.rx_request()
            .mapArray(Cat.self)
            // Now we do retry in the flatMap, why?
            // Because this way it will start from the Api request
            // and not the logic before the flatMap.
            .retryWhen { errors in
                return errors
                    .map { error -> Error in
                        switch error {
                        case is URLError: return error
                        default: throw error
                        }
                    }
                    .take(2)
            }
            .catchErrorJustReturn([])
    }
    ...
```

Quick explanation. The parameter in `retryWhen` is an observable sequence of `Error` type. Now when we return the sequence and a new item in this sequence is produced, the original sequence is started once again. When the error sequence completes/errors out, the original sequence will complete/error out as well. So we complete our error sequence after 2 `URLError` errors or we throw an error whenever different error comes. This means: either you retry maximum 2 times when `URLError` comes or just propagate the error down below.

**_Warnings:_**<br />
👮 The same as before (with `retry(_:)`), remember that error can happen after the `retryWhen(_:)` block and you should protect yourself as well.

## 3. Driver

## 4. Result
