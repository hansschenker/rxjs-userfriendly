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

## buffer*, window*, and debounce*

These three are often confused because each can be given a duration. They answer different questions.

`buffer*` collects. `batchEvery(1000)` keeps every value and emits one array when the second ends. Nothing is dropped. You see the values only when the batch closes.

`window*` splits. `spanEvery(1000)` emits an observable for that same second. Subscribe to it and values arrive as they happen. Same clock as `batchEvery`, different result: a live stream, not the closed list.

`debounce*` waits for quiet. `afterQuiet(1000)` emits a single value, the latest, and only after 1000 ms with no new value. Values that arrived during the wait are discarded, not collected. A steady stream faster than the duration emits nothing until it pauses.

| | What you receive | Values kept | Clock starts |
| --- | --- | --- | --- |
| `batch*` (`buffer*`) | one `T[]` when the span closes | all of them | duration, count, or signal |
| `span*` (`window*`) | an `Observable<T>` while the span is open | all of them, as they arrive | duration, count, or signal |
| `afterQuiet*` (`debounce*`) | one `T` after silence | the latest only | each new source value resets it |

`laterBy` (`delay`) is neither. It shifts every value and drops nothing.

## Names

| Name | Operator | Summary |
| --- | --- | --- |
| `firstEvery(ms)` | `throttleTime` | Emit the first value, then ignore for `ms`. |
| `firstAndLastEvery(ms)` | `throttleTime`, both edges | Emit the opener, and the last value if a later one arrived. |
| `firstWhen(signal$)` | `throttle` | Emit the opener, then ignore until `signal$` emits. |
| `lastEvery(ms)` | `auditTime` | Hold values for `ms`, then emit the latest. |
| `lastWhen(signal$)` | `audit` | Hold values until `signal$` emits, then emit the latest. |
| `lastOnFrame()` | `auditTime(0, animationFrameScheduler)` | The next paint is the window. Emit the latest value before it. |
| `pollEvery(ms)` | `sampleTime` | On a fixed clock, emit the latest value if one arrived. |
| `pollWhen(signal$)` | `sample` | When `signal$` emits, emit the latest value if one arrived. |
| `afterQuiet(ms)` | `debounceTime` | Emit the latest value only after `ms` with nothing new. |
| `afterQuietWhen(fn)` | `debounce` | Each value picks the signal that ends its quiet wait. |
| `laterBy(ms)` | `delay` | Shift each value by `ms`, or until a date. Drop nothing. |
| `laterWhen(fn)` | `delayWhen` | Each value waits for its own signal. |
| `failAfter(ms)` | `timeout` | Error if the next value does not arrive in time. |
| `withGap()` | `timeInterval` | Add the time since the previous value. |
| `withTime()` | `timestamp` | Add the time the value was emitted. |
| `batchEvery(ms)` | `bufferTime` | Emit an array of every value in the duration. |
| `batchOf(n)` | `bufferCount` | Emit an array of every `n` values. |
| `batchWhen(signal$)` | `buffer` | Close the array when `signal$` emits, then start another. |
| `batchUntil(fn)` | `bufferWhen` | Each batch asks `fn` for the signal that closes it. |
| `batchBetween(open$, closeFn)` | `bufferToggle` | Open an array on `open$`, close it with `closeFn`. |
| `spanEvery(ms)` | `windowTime` | Emit an observable for each duration. |
| `spanOf(n)` | `windowCount` | Emit an observable for every `n` values. |
| `spanWhen(signal$)` | `window` | Close the observable when `signal$` emits, then start another. |
| `spanUntil(fn)` | `windowWhen` | Each span asks `fn` for the signal that closes it. |
| `spanBetween(open$, closeFn)` | `windowToggle` | Open an observable on `open$`, close it with `closeFn`. |
| `shared()` | `share` | One upstream while anyone listens. Restart after idle. |
| `sharedLast()` | `shareReplay({ bufferSize: 1, refCount: true })` | Late subscribers get the latest value, then live values. |
| `sharedRecent(n)` | `shareReplay({ bufferSize: n, refCount: true })` | Late subscribers get the last `n` values. |
| `sharedRecent(n, ms)` | `shareReplay` with `windowTime` | Replay buffer also drops values older than `ms`. |
| `sharedFinal()` | `share` with `AsyncSubject` | Emit only the last value, and only on complete. |
| `sharedUntilQuiet(ms)` | `share` with a reset timer | Keep the upstream for `ms` after the last subscriber leaves. |
| `connected()` | `connectable` / `publish` | Subscribers wait until `connect()`. |
| `connectedLast(seed)` | `connectable` + `BehaviorSubject` | Manual connect, late subscribers get the seed or latest value. |
| `connectedRecent(n)` | `connectable` + `ReplaySubject` | Manual connect, replay the last `n`. |
| `connectedRecent(n, ms)` | `publishReplay` | Manual connect, replay aged by `ms`. |
| `connectedFinal()` | `connectable` + `AsyncSubject` | Manual connect, emit the last value on complete. |
| `runAll(fn)` | `mergeMap` | Start every inner. Emit as they arrive. |
| `runLatest(fn)` | `switchMap` | Unsubscribe the previous inner. Keep the newest. |
| `runInOrder(fn)` | `concatMap` | Queue inners. Start the next when the current completes. |
| `runUnlessBusy(fn)` | `exhaustMap` | Drop new source values while an inner is active. |
| `as(fn)` | `map` | Replace each value. |
| `keep(pred)` | `filter` | Pass values that match. |
| `peek(fn)` | `tap` | Look at a value without changing the stream. |
| `onEnd(fn)` | `finalize` | Run when the subscription ends, for any reason. |
| `onlyFirst()` | `first` | Emit one value, then complete. |
| `onlyLast()` | `last` | Emit the last value on complete. |
| `exactlyOne()` | `single` | Error unless the source emits exactly one match. |
| `at(index)` | `elementAt` | Emit the value at that index, or error. |
| `firstMatch(pred)` | `find` | Emit the first value that matches, then complete. |
| `take(n)` | `take` | Emit the first `n` values, then complete. |
| `takeLast(n)` | `takeLast` | Emit the last `n` values on complete. |
| `takeWhile(pred)` | `takeWhile` | Emit while `pred` holds, then complete. |
| `until(signal$)` | `takeUntil` | Complete when `signal$` emits. |
| `skip(n)` | `skip` | Drop the first `n` values. |
| `skipLast(n)` | `skipLast` | Drop the last `n` values. |
| `skipWhile(pred)` | `skipWhile` | Drop values while `pred` holds. |
| `after(signal$)` | `skipUntil` | Drop values until `signal$` emits. |
| `skipSame()` | `distinctUntilChanged` | Drop a value equal to the previous one. |
| `skipSameBy(key)` | `distinctUntilKeyChanged` | Drop a value whose key equals the previous key. |
| `skipSeen()` | `distinct` | Drop values already seen. |
| `beginWith(...values)` | `startWith` | Emit these values first. |
| `finishWith(...values)` | `endWith` | Emit these values on complete. |
| `ifEmpty(value)` | `defaultIfEmpty` | Emit `value` if the source completed empty. |
| `failIfEmpty()` | `throwIfEmpty` | Error if the source completed empty. |
| `dropValues()` | `ignoreElements` | Emit nothing. Pass complete and error. |
| `withPrevious()` | `pairwise` | Emit `[previous, current]`. |
| `running(fn, seed)` | `scan` | Emit each step of a fold. |
| `fold(fn, seed)` | `reduce` | Emit the fold once, on complete. |
| `collect()` | `toArray` | Emit every value as one array on complete. |
| `count()` | `count` | Emit how many values passed. |
| `least()` | `min` | Emit the smallest value on complete. |
| `greatest()` | `max` | Emit the largest value on complete. |
| `onError(fn)` | `catchError` | Replace the error with another stream. |
| `retry(n)` | `retry` | Resubscribe after an error. |
| `repeat(n)` | `repeat` | Resubscribe after complete. |
| `again(fn)` | `expand` | Feed each output back through `fn`. |
| `groupBy(key)` | `groupBy` | Split into one observable per key. |
| `withLatest(other$)` | `withLatestFrom` | Pair each value with the latest from `other$`. |
| `pairedWith(other$)` | `zipWith` | Pair values by index. Wait for both. |
| `latestOf(other$)` | `combineLatestWith` | Emit when either side emits, with the latest of each. |
| `then(other$)` | `concatWith` | Subscribe to `other$` after this source completes. |
| `alongWith(other$)` | `mergeWith` | Merge this source with `other$`. |
| `firstToEmit(other$)` | `raceWith` | Keep whichever source emits first. |
| `whenAllDone(others)` | `forkJoin` | Emit the last value of each source when all complete. |
| `emitOn(scheduler)` | `observeOn` | Deliver notifications on that scheduler. |
| `subscribeOn(scheduler)` | `subscribeOn` | Call subscribe on that scheduler. |
| `notices()` | `materialize` | Turn next, error, and complete into notice values. |
| `valuesFromNotices()` | `dematerialize` | Turn notices back into notifications. |

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
