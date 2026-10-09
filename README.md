# rxjs-userfriendly

User-friendly names for technical RxJS operator names.

The name states the policy. The technical name stays in a comment, because a friendly name that hides a real difference is worse than the original.

The main contributor to this project is SuperGrok.

## Grammar

- `Every` is a duration you chose.
- `When` is a signal that ends or triggers it.
- `first` and `last` are which value survives.
- `after` shifts a value, or waits for a signal before values pass.
- `with` adds data. It does not drop values.
- `batch` emits an array when the span closes.
- `span` emits an observable while the span is open.
- `shared` starts on the first subscriber and stops when the last one leaves.
- `connected` does nothing until `connect()`, and does not stop when subscribers leave.
- `run*` is the flattening policy: what happens to the previous inner subscription.

Each name wraps one operator. `lastEvery` stays on `auditTime` and does not also mean trailing `throttleTime`.

## Time

```ts
firstEvery(ms)          // throttleTime, leading
lastEvery(ms)           // auditTime
firstAndLastEvery(ms)   // throttleTime, both edges
lastOnFrame()           // auditTime(0, animationFrameScheduler)

firstWhen(signal$)      // throttle(signal$)
lastWhen(signal$)       // audit(signal$)

afterQuiet(ms)          // debounceTime
afterQuietWhen(fn)      // debounce

pollEvery(ms)           // sampleTime
pollWhen(signal$)       // sample

laterBy(ms)             // delay
laterWhen(fn)           // delayWhen
failAfter(ms)           // timeout

withGap()               // timeInterval
withTime()              // timestamp

batchEvery(ms)          // bufferTime
batchWhen(signal$)      // buffer
spanEvery(ms)           // windowTime
spanWhen(signal$)       // window
```

`lastEvery` starts its window from a source value and emits when that window ends. `pollEvery` ticks on a fixed clock and emits nothing if no new value arrived.

`spanEvery(1000)` emits an `Observable<T>` per second. `batchEvery(1000)` emits the `T[]` when that second ends.

Do not point `lastEvery` at `throttleTime(..., { leading: false, trailing: true })`. For a plain duration those two are almost the same, but they are not the same operator.

## Multicast

`publish*` is the manual-connect form of `share*`. RxJS 7 deprecates `publish*` in favor of `share` and `connectable`.

```ts
shared()                 // share()
sharedLast()             // shareReplay({ bufferSize: 1, refCount: true })
sharedRecent(n)          // shareReplay({ bufferSize: n, refCount: true })
sharedFinal()            // share with an AsyncSubject

connected()              // connectable(source) / publish()
connectedLast(seed)      // connectable + BehaviorSubject / publishBehavior
connectedRecent(n)       // connectable + ReplaySubject / publishReplay
connectedFinal()         // connectable + AsyncSubject / publishLast
```

`shared()` emits nothing to a subscriber who arrives late. `sharedLast()` gives that subscriber the latest value, then live values. `sharedFinal()` stays silent until the source completes, then emits that one value.

Do not point `sharedLast` at bare `shareReplay(1)`. That old signature never unsubscribes from the source. The `refCount: true` form is the one that stops when idle.

`connect()` on a `connected` observable is a method. It subscribes the inner subject to the source once. A second call while that connection is open is idempotent. Unsubscribing the returned subscription disconnects every consumer at once. The `connect` operator is a different function: it ties that connection to the subscription of whatever the selector returns.

## Flattening

```ts
runAll(fn)            // mergeMap
runLatest(fn)         // switchMap
runInOrder(fn)        // concatMap
runUnlessBusy(fn)     // exhaustMap

runAll()              // mergeAll
runLatest()           // switchAll
runInOrder()          // concatAll
runUnlessBusy()       // exhaustAll
```

`runAll(fn, 2)` is the `mergeMap` concurrency limit. `switchMapTo`, `mergeMapTo`, and `concatMapTo` are retired. A constant inner is `runLatest(() => inner$)`.

`again(fn)` is `expand`. It does not fit this set: it resubscribes the projection to its own output.

## Values, limits, and joins

```ts
as(fn)                  // map
keep(pred)              // filter
peek(fn)                // tap
onEnd(fn)               // finalize

onlyFirst()             // first
onlyFirst(pred)         // first(pred)
onlyLast()              // last
exactlyOne()            // single
at(index)               // elementAt
firstMatch(pred)        // find

take(n)                 // take
takeLast(n)             // takeLast
takeWhile(pred)         // takeWhile
until(signal$)          // takeUntil

skip(n)                 // skip
skipLast(n)             // skipLast
skipWhile(pred)         // skipWhile
after(signal$)          // skipUntil

skipSame()              // distinctUntilChanged
skipSameBy(key)         // distinctUntilKeyChanged
skipSeen()              // distinct

beginWith(...values)    // startWith
finishWith(...values)   // endWith
ifEmpty(value)          // defaultIfEmpty
failIfEmpty()           // throwIfEmpty
dropValues()            // ignoreElements

withPrevious()          // pairwise
running(fn, seed)       // scan
fold(fn, seed)          // reduce
collect()               // toArray
count()                 // count
least()                 // min
greatest()              // max

onError(fn)             // catchError
retry(n)                // retry
repeat(n)               // repeat
again(fn)               // expand

groupBy(key)            // groupBy
batchOf(n)              // bufferCount
spanOf(n)               // windowCount

withLatest(other$)      // withLatestFrom
pairedWith(other$)      // zipWith
latestOf(other$)        // combineLatestWith
then(other$)            // concatWith
alongWith(other$)       // mergeWith
firstToEmit(other$)     // raceWith
whenAllDone(others)     // forkJoin

emitOn(scheduler)       // observeOn
subscribeOn(scheduler)  // subscribeOn

notices()               // materialize
valuesFromNotices()     // dematerialize
```

`onlyFirst()` is the `first` operator: one value, then complete. `firstEvery(ms)` is still the leading throttle. `after(signal$)` waits for a signal before letting values through. `laterBy(ms)` is still the delay.
