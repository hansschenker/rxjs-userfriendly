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

## throttle*

```ts
firstEvery(ms)                 // throttleTime(ms), leading only, the default
firstEvery(ms, scheduler)      // throttleTime(ms, scheduler)
firstAndLastEvery(ms)          // throttleTime(ms, scheduler, { leading: true, trailing: true })
firstAndLastEvery(ms, scheduler)

firstWhen(signal$)             // throttle(signal$), leading only
firstWhen(signal$, { leading: true, trailing: true })
```

`firstEvery(1000)` emits the value that opens the window, then drops values until that duration ends. `firstAndLastEvery(1000)` also emits the latest value from inside the window when it ends. In RxJS 7 that trailing emit starts a new silent interval.

There is no friendly alias for trailing-only `throttleTime`. That shape is `lastEvery`, and `lastEvery` wraps `auditTime`, not `throttleTime`.

## sample*

```ts
pollEvery(ms)                  // sampleTime(ms)
pollEvery(ms, scheduler)       // sampleTime(ms, scheduler)
pollWhen(signal$)              // sample(signal$)
```

The clock is independent of the source. On each tick, emit the latest value if one arrived since the previous tick, otherwise emit nothing. `pollEvery` is not `lastEvery`: `lastEvery` opens its window from a source value and always ends with an emit.

## debounce*

```ts
afterQuiet(ms)                 // debounceTime(ms)
afterQuiet(ms, scheduler)      // debounceTime(ms, scheduler)
afterQuietWhen(fn)             // debounce(fn)
```

Every new value resets the wait. The latest value is emitted only after `ms` with nothing new, or after the signal chosen by `fn` emits. A value every 200 ms through `afterQuiet(1000)` emits nothing until the source goes quiet.

## delay*

```ts
laterBy(ms)                    // delay(ms)
laterBy(ms, scheduler)         // delay(ms, scheduler)
laterBy(date)                  // delay(date)
laterWhen(fn)                  // delayWhen(fn)
```

Each value is shifted. Order is kept. Nothing is dropped. `laterBy` waits a duration or until a date. `laterWhen` lets each value pick its own delay signal. `after(signal$)` is not this family: that name is `skipUntil`.

## buffer*

`batch` emits `T[]` when the span closes.

```ts
batchEvery(ms)                 // bufferTime(ms)
batchEvery(ms, { every })      // bufferTime(ms, every)
batchEvery(ms, { max })        // bufferTime(ms, undefined, max)
batchEvery(ms, scheduler)      // bufferTime with a scheduler

batchOf(n)                     // bufferCount(n)
batchOf(n, { every })          // bufferCount(n, every)

batchWhen(signal$)             // buffer(signal$)
batchUntil(fn)                 // bufferWhen(fn)
batchBetween(open$, closeFn)   // bufferToggle(open$, closeFn)
```

`batchWhen` closes the current array when `signal$` emits, then starts another. `batchUntil` asks `fn` for a fresh closing signal each time a batch opens. `batchBetween` opens an array when `open$` emits and closes it when `closeFn` for that opening emits. Overlapping openings produce overlapping arrays.

## window*

`span` emits `Observable<T>` while the span is open. Same clocks as `batch`, different result: a live stream, not the closed list.

```ts
spanEvery(ms)                  // windowTime(ms)
spanEvery(ms, { every })       // windowTime(ms, every)
spanEvery(ms, { max })         // windowTime(ms, undefined, max)
spanEvery(ms, scheduler)       // windowTime with a scheduler

spanOf(n)                      // windowCount(n)
spanOf(n, { every })           // windowCount(n, every)

spanWhen(signal$)              // window(signal$)
spanUntil(fn)                  // windowWhen(fn)
spanBetween(open$, closeFn)    // windowToggle(open$, closeFn)
```

`spanEvery(1000)` emits an observable per second; subscribe to each one to see values as they arrive. `batchEvery(1000)` emits one array when that second ends.

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
sharedRecent(n, ms)      // shareReplay({ bufferSize: n, windowTime: ms, refCount: true })
sharedFinal()            // share with an AsyncSubject
sharedUntilQuiet(ms)     // share({ resetOnRefCountZero: () => timer(ms) })

connected()              // connectable(source) / publish()
connectedLast(seed)      // connectable + BehaviorSubject / publishBehavior
connectedRecent(n)       // connectable + ReplaySubject / publishReplay
connectedRecent(n, ms)   // ReplaySubject buffer aged by ms / publishReplay
connectedFinal()         // connectable + AsyncSubject / publishLast
```

`shared()` emits nothing to a subscriber who arrives late. `sharedLast()` gives that subscriber the latest value, then live values. `sharedFinal()` stays silent until the source completes, then emits that one value. `sharedUntilQuiet(ms)` keeps the upstream alive for `ms` after the last subscriber leaves.

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
