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

These three can all take a duration. They do not do the same job.

`buffer*` collects. Every value in the span is kept. You receive one array when the span closes, and nothing before that.

`window*` splits. Every value is kept too, but you receive an observable when the span opens. Values come out of that observable as they arrive. Same clock as `buffer*`, different shape: a live stream, not the closed list.

`debounce*` waits for quiet. It does not collect. Each new value throws away the previous one and restarts the timer. Only the latest value is emitted, and only after the quiet period. A stream that never pauses emits nothing.

Same source, three results. The window is three ticks. `-` is silence.

```text
source          a--b--c--d--e--f--|

batchEvery      ---------[a,b,c]---------[d,e,f]|
spanEvery       +----------------+----------------+
                a--b--c|         d--e--f|

afterQuiet      -------------------------------f|
```

`batchEvery` emits `[a,b,c]` at the first boundary and `[d,e,f]` at the second. `spanEvery` emits two observables; the first produces `a`, `b`, `c` as they happen. `afterQuiet` emits only `f`, because every earlier value was followed by another before the quiet period ended.

| | What you receive | Values kept | Clock |
| --- | --- | --- | --- |
| `batch*` (`buffer*`) | one `T[]` when the span closes | all | duration, count, or signal |
| `span*` (`window*`) | an `Observable<T>` while the span is open | all, as they arrive | duration, count, or signal |
| `afterQuiet*` (`debounce*`) | one `T` after silence | the latest only | each value resets it |

`laterBy` (`delay`) is neither. It shifts every value and drops nothing.

### batch* visuals

`batchEvery(3)` closes an array every three ticks. `bufferTime`.

```text
source      a--b--c--d--e--f--|
batchEvery  ---------[a,b,c]---------[d,e,f]|
```

`batchOf(2)` closes an array every two values. `bufferCount`.

```text
source   a--b--c--d--|
batchOf  ---[a,b]---[c,d]|
```

`batchWhen(signal$)` closes when the signal emits, then starts again. `buffer`.

```text
source     a--b--c--d--e--|
signal     ------x--------x
batchWhen  ------[a,b]----[c,d,e]|
```

`batchUntil(fn)` asks for a new closing signal each time a batch opens. `bufferWhen`.

```text
source      a--b--c-----d--e--|
close       ------x-----------x
batchUntil  ------[a,b]-------[c,d,e]|
```

`batchBetween(open$, closeFn)` opens on `open$` and closes with the signal for that opening. `bufferToggle`. Overlaps stay overlaps.

```text
source        a--b--c--d--e--|
open          x--------x
batchBetween  ---[a,b,c]--[d,e]|
```

### span* visuals

`spanEvery(3)` emits an observable per three ticks. `windowTime`. The inner line is what a subscriber of that span sees.

```text
source     a--b--c--d--e--f--|
spanEvery  +----------------+----------------+
           a--b--c|         d--e--f|
```

`spanOf(2)` emits an observable per two values. `windowCount`.

```text
source  a--b--c--d--|
spanOf  +-----+-----+
        a--b| c--d|
```

`spanWhen(signal$)` closes the current observable when the signal emits. `window`.

```text
source    a--b--c--d--e--|
signal    ------x--------x
spanWhen  +-----+--------+
          a--b| c--d--e|
```

`spanUntil(fn)` asks for a fresh closing signal per span. `windowWhen`.

```text
source     a--b--c-----d--e--|
spanUntil  +-----+-----------+
           a--b| c-----d--e|
```

`spanBetween(open$, closeFn)` opens an observable on `open$`. `windowToggle`.

```text
source       a--b--c--d--e--|
open         x--------x
spanBetween  +--------+-----+
             a--b--c| d--e|
```

### afterQuiet* visuals

`afterQuiet(3)` emits the latest value only after three quiet ticks. `debounceTime`. Here `c` and `f` each get a quiet gap.

```text
source      a-b-c-----d-e-f--|
afterQuiet  ------c----------f|
```

A source that never pauses emits nothing:

```text
source      a-b-c-d-e-f-|
afterQuiet  -------------|
```

`afterQuietWhen(fn)` is the same wait, but each value picks the signal that ends it. `debounce`.

```text
source          a----b----c--|
quiet for a     ------x
quiet for b          ----x
quiet for c               --x
afterQuietWhen  ------a----b--c|
```

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

## Contributors

The main contributor to this project is SuperGrok (`supergrok@x.ai`).
