# rxjs-userfriendly

User-friendly names for technical RxJS operator names. Status: proposed vocabulary, ready for review.

The name states the policy. The technical name stays in a comment, because a friendly name that hides a real difference is worse than the original.

The main contributor to this project is SuperGrok.

## Grammar

A name is a root, zero or more suffix tokens, and an argument. Every token has exactly one meaning and lives in exactly one slot. Every slot answers one question of the operator policy model in `rxjs-policy-debugger`, so the name states the policy and nothing else.

Status: extended grammar, applied to the Names table and to `index.html`. The names it replaced are listed under "Renamed from the first draft".

The closed token list. One line per token. The tables that follow give each token its policy, its argument, and the names it builds.

Roots:

- `first` and `last` are which value survives. `last` is the latest so far.
- `poll` emits the latest value on a clock or a signal, if one arrived.
- `afterQuiet` emits the latest value once the source has been quiet.
- `later` shifts each value in time. It drops nothing.
- `after` waits for a signal before values pass.
- `until` completes the stream when a signal fires once.
- `fail` errors the stream.
- `with` adds data to each value. It does not drop values. The suffix names the data: `Gap`, `Time`, `Previous`, `Latest`.
- `batch` emits an array when the span closes.
- `span` emits an observable while the span is open.
- `run*` maps each value to inner work and flattens it. The suffix is the concurrency policy.
- `flatten*` subscribes to the streams the source already emits. Same suffixes as `run*`.
- `accumulate` folds values into state and emits the state after each value.
- `fold`, `collect`, `count`, `least`, `greatest` fold everything and emit once, at completion.
- `as` replaces each value.
- `keep` passes values that match.
- `only*` passes only this: `onlyFirst`, `onlyFinal`, `onlyEnd`.
- `peek` looks and changes nothing.
- `on*` runs a callback at a lifecycle point: `onEnd`, `onError`.
- `shared` starts on the first subscriber and stops when the last one leaves.
- `connected` does nothing until `connect()`, and does not stop when subscribers leave.
- `begin` and `finish` add literal values before the first value or after the last.
- `then` subscribes to the next source after this one ends.
- `skip`, `take`, `retry`, `repeat`, `groupBy`, `isEmpty`, `observeOn`, `subscribeOn` keep their technical names.

Trigger and time suffixes:

- `Every` is a duration you chose, recurring. `EveryFrame` is the animation frame.
- `Of` is a count of values.
- `On` is one external signal. Every firing triggers.
- `When` is a function you supply. It returns a fresh signal per value or span, and that signal fires once.
- `Until` is one external signal. It fires once and ends the stream.
- `Between` opens on a signal and closes on the signal made for that opening.
- `While` lasts as long as a predicate holds.
- `After` waits for a duration or a signal first. Same meaning as the root `after`.
- `Quiet` is a duration with no source value.

Concurrency suffixes, one per policy of `rxjs-policy-debugger`:

- `Concurrent` lets inners overlap. Nothing is cancelled. `allowConcurrent`.
- `InOrder` runs one inner at a time and queues the rest. `queueWhileBusy`.
- `Latest` runs the newest inner and cancels the previous one. `keepLatest`.
- `UnlessBusy` runs the first inner and drops new values while it runs. `ignoreWhileBusy`.
- `Recursive` feeds every output back in.
- `Combined` and `Paired` are the join policies of `flatten`: latest of each, or by index.

Sharing suffixes:

- `Last` replays the latest value to late subscribers.
- `Recent` replays the last `n`, or only those younger than `ms`.
- `Final` is the value at completion.
- `Linger` keeps the upstream for `ms` after the last subscriber leaves.
- `Inside` builds the pipeline that uses the shared source.

Ending tokens:

- `End` is the end of the subscription: complete, error, or unsubscribe.
- `IfEmpty` is a source that completed with no value.
- `IfQuiet` is no value within `ms`.

Comparison suffixes:

- `Same` is equal to the previous value, or to another source.
- `Seen` is equal to any earlier value.
- `By` compares or groups by a key.
- `Match` satisfies a predicate.

Position rule, the only one: prefix `with` attaches data, suffix `With` joins another source or literal values.

### Slots

| Slot | Policy question it answers | Token group |
| --- | --- | --- |
| Root | Value and Cardinality: what comes out, and how many per input | Roots |
| Trigger suffix | Trigger and Time: what causes output, and when | Trigger and time |
| Concurrency suffix | Concurrency and Cancellation: what happens to inner work that is still running | Concurrency |
| Sharing root and suffix | Sharing, Connection, Disconnection, Replay, Reset | Sharing |
| Ending token | Termination: what happens on complete and on error | Ending |
| Comparison suffix | Value and Cardinality: which values count as the same, or match | Comparison |
| Argument | Source and Initialization: what feeds the operator, and the state it starts with | Arguments |

Type policies are not in names. TypeScript carries them.

### Roots

A root in lower case is the first word. The same word in upper case is a suffix with the same meaning (`last` in `lastEvery`, `Last` in `sharedLast`).

| Root | Means | Names |
| --- | --- | --- |
| `first` | the first value of a window or race survives | `firstEvery`, `firstWhen`, `firstAndLastEvery`, `firstMatch`, `firstToEmit`, `onlyFirst` |
| `last` | the latest value so far survives | `lastEvery`, `lastWhen`, `lastEveryFrame`, `sharedLast`, `connectedLast` |
| `poll` | on a clock or a signal, emit the latest value if one arrived | `pollEvery`, `pollOn` |
| `afterQuiet` | emit the latest value once the source has been quiet | `afterQuiet`, `afterQuietWhen` |
| `later` | shift each value in time, drop nothing | `later`, `laterWhen` |
| `after` | values pass only after a signal | `after` |
| `until` | the stream completes when a signal fires once | `until` |
| `fail` | error the stream | `failIfQuiet`, `failIfEmpty` |
| `with` (prefix) | attach data to each value, drop nothing | `withGap`, `withTime`, `withPrevious`, `withLatest` |
| `batch` | collect values, emit one array when the span closes | `batchEvery`, `batchOf`, `batchOn`, `batchWhen`, `batchBetween` |
| `span` | split values, emit one observable while the span is open | `spanEvery`, `spanOf`, `spanOn`, `spanWhen`, `spanBetween` |
| `run` | map each value to inner work and flatten it | `runConcurrent`, `runInOrder`, `runLatest`, `runUnlessBusy`, `runRecursive` |
| `flatten` | the source already emits streams; subscribe to them | `flattenConcurrent`, `flattenInOrder`, `flattenLatest`, `flattenUnlessBusy`, `flattenCombined`, `flattenPaired` |
| `accumulate` | fold values into state, emit the state after each value | `accumulate`, `accumulateConcurrent`, `accumulateLatest` |
| `fold`, `collect`, `count`, `least`, `greatest` | aggregate roots: fold everything, emit once at completion | `fold`, `collect`, `count`, `least`, `greatest` |
| `as` | replace each value | `as`, `asNotices` |
| `keep` | pass values that match | `keep` |
| `only` | only this passes | `onlyFirst`, `onlyFinal`, `onlyEnd` |
| `peek` | look, change nothing | `peek` |
| `on` (prefix) | run a callback at a lifecycle point | `onEnd`, `onError` |
| `shared` | one upstream; starts with the first subscriber, stops with the last | `shared`, `sharedLast`, `sharedRecent`, `sharedFinal`, `sharedLinger`, `sharedInside` |
| `connected` | one upstream; nothing until `connect()`, does not stop when subscribers leave | `connected`, `connectedLast`, `connectedRecent`, `connectedFinal` |
| `begin`, `finish` | literal values before the first value, or after the last | `beginWith`, `finishWith` |
| `then` | after this source ends, subscribe to the next one | `then`, `thenEvenIfFailed` |
| `skip`, `take`, `retry`, `repeat`, `groupBy`, `isEmpty`, `observeOn`, `subscribeOn` | kept technical: already plain English and grammar-conform | `skip`, `skipLast`, `skipWhile`, `skipSame`, `skipSameBy`, `skipSeen`, `skipSeenBy`, `take`, `takeLast`, `takeWhile`, `retry`, `retryAfter`, `repeat`, `repeatAfter` |

Suffix `With` is the one position rule: prefix `with` attaches data, suffix `With` joins another source or literal values to this one (`alongWith`, `pairedWith`, `combinedWith`, `beginWith`, `finishWith`).

### Trigger and time

| Token | Argument | Means | Names |
| --- | --- | --- | --- |
| `Every` | `ms` | a fixed clock you chose, recurring | `firstEvery`, `lastEvery`, `pollEvery`, `batchEvery`, `spanEvery` |
| `EveryFrame` | none | the animation frame is the clock | `lastEveryFrame` |
| `Of` | `n` | a count of values | `batchOf`, `spanOf` |
| `On` | `signal$` | one external signal; every firing triggers | `pollOn`, `batchOn`, `spanOn` |
| `When` | `fn` | you make the signal, fresh for each value or span; it fires once | `firstWhen`, `lastWhen`, `afterQuietWhen`, `laterWhen`, `batchWhen`, `spanWhen` |
| `Until` | `signal$` | one external signal; fires once and ends the stream | `until` |
| `Between` | `open$, fn` | opens on `open$`, closes on the signal `fn` makes for that opening; overlaps stay overlaps | `batchBetween`, `spanBetween` |
| `While` | `pred` | as long as `pred` holds | `takeWhile`, `skipWhile` |
| `After` | `ms` or `fn` | wait this long, or for this signal, first; same meaning as the root `after` | `retryAfter`, `repeatAfter` |
| `Quiet` | `ms` | `ms` with no source value | `afterQuiet`, `failIfQuiet`, `ifQuiet` |

`On`, `When`, `Until` are three different shapes, not three spellings. `On` is one signal that keeps firing. `When` is a function you supply, called per value or per span, returning a signal that fires once. `Until` is one signal that fires once and ends everything. `throttle`, `audit`, `debounce`, `delayWhen`, `bufferWhen`, `windowWhen` all take a function, so they are all `When`.

### Concurrency

The four policies of `rxjs-policy-debugger`, as suffixes. The same four words serve every family that has a concurrency axis: `run*`, `flatten*`, `accumulate*`.

| Token | Inner work | Cancellation | Policy vocabulary | Names |
| --- | --- | --- | --- | --- |
| `Concurrent` | all inners may overlap | nothing is cancelled | `allowConcurrent` | `runConcurrent`, `flattenConcurrent`, `accumulateConcurrent` |
| `InOrder` | one at a time; new values wait in a queue | nothing is cancelled, order is kept | `queueWhileBusy` | `runInOrder`, `flattenInOrder` |
| `Latest` | one at a time; always the newest | a new value cancels the previous inner | `keepLatest` | `runLatest`, `flattenLatest`, `accumulateLatest` |
| `UnlessBusy` | one at a time; the first | new values are dropped while busy | `ignoreWhileBusy` | `runUnlessBusy`, `flattenUnlessBusy` |
| `Recursive` | every output is fed back in, concurrently | nothing is cancelled | none | `runRecursive` |
| `Combined`, `Paired` | join policies for `flatten`: latest of each, or by index | none | none | `flattenCombined`, `flattenPaired` |

### Sharing

| Token | Policy | Means | Names |
| --- | --- | --- | --- |
| `shared` | Sharing, Connection, Disconnection, Reset | one upstream; connects on the first subscriber, disconnects on the last, resets after | `shared` |
| `connected` | Sharing, Connection | one upstream; connects on `connect()`, never disconnects on its own | `connected` |
| `Last` | Replay | late subscribers get the latest value, then live values | `sharedLast`, `connectedLast(seed)` |
| `Recent` | Replay | late subscribers get the last `n`; with `ms`, only values younger than `ms` | `sharedRecent(n)`, `sharedRecent(n, ms)`, `connectedRecent` |
| `Final` | Replay, Termination | everyone gets only the value at completion | `sharedFinal`, `connectedFinal` |
| `Linger` | Disconnection | keep the upstream `ms` after the last subscriber leaves | `sharedLinger(ms)` |
| `Inside` | Connection | `fn` builds the pipeline that uses the shared source; connects on subscribe | `sharedInside(fn)` |

### Ending

| Token | Means | Names |
| --- | --- | --- |
| `Final` | the value at completion | `onlyFinal`, `finalOfEach`, `sharedFinal`, `connectedFinal` |
| `End` | the end of the subscription: complete, error, or unsubscribe | `onEnd`, `onlyEnd` |
| `IfEmpty` | the source completed with no value | `ifEmpty(value)`, `failIfEmpty()` |
| `IfQuiet` | no value arrived within `ms` | `failIfQuiet(ms)`, `ifQuiet(ms, other$)` |
| `onError` | the error is replaced by another stream | `onError(fn)` |
| `then` | this source ended, the next one starts | `then`, `thenEvenIfFailed` |

Aggregate roots (`fold`, `collect`, `count`, `least`, `greatest`) and `onlyFinal`, `takeLast`, `skipLast` emit at completion by definition. No suffix says so.

### Comparison

| Token | Argument | Means | Names |
| --- | --- | --- | --- |
| `Same` | none, or `other$` | equal to the previous value, or to another source | `skipSame`, `skipSameBy`, `sameAs` |
| `Seen` | none | equal to any earlier value | `skipSeen`, `skipSeenBy` |
| `By` | `key` | compare or group by a key | `skipSameBy`, `skipSeenBy`, `groupBy` |
| `Match` | `pred` | satisfies a predicate | `firstMatch`, `firstMatchIndex`, `allMatch` |

### Arguments

| Argument | Shape | Policy |
| --- | --- | --- |
| `ms` | milliseconds; `later` also takes a `Date` | Time |
| `n` | a count | Cardinality |
| `signal$` | an observable; only its timing matters, never its values | Trigger |
| `fn` | a function; what it returns follows the root: inner work for `run*`, a closing signal for `When`, a replacement for `as`, a handler for `on*`, the next state for `accumulate` and `fold` | Source |
| `pred` | `(value) => boolean` | Cardinality |
| `key` | `(value) => key`, or a property name | Value |
| `seed` | the state before the first value | Initialization |
| `other$` | one other source | Source |
| `sources` | several sources | Source |
| `open$` | the opening signal of `Between` | Trigger |
| `value`, `...values` | literal values | Source |
| `scheduler` | only with the kept technical names `observeOn`, `subscribeOn` | Time |

### Rules

1. One token, one meaning. The only position rule is prefix `with` versus suffix `With`.
2. One name, one policy. Argument type or arity picks the RxJS operator: `sharedRecent(n)` and `sharedRecent(n, ms)`, `retryAfter(ms)` and `retryAfter(fn)`.
3. A technical name is kept when it is already plain English and obeys the grammar.
4. Aggregate roots emit once, at completion. No suffix.
5. A friendly name never hides a real difference. `lastEvery` stays on `auditTime` and does not also mean trailing `throttleTime`.
6. Scope is the pipeable operators of RxJS 7.8. Deprecated operators get no name; their modern replacement does.

### Alignment with the policy vocabulary

The canonical vocabulary operators of `rxjs-policy-debugger` and the friendly names say the same policy.

| Policy vocabulary | RxJS | Friendly name |
| --- | --- | --- |
| `allowConcurrent` | `mergeMap` | `runConcurrent(fn)` |
| `queueWhileBusy` | `concatMap` | `runInOrder(fn)` |
| `keepLatest` | `switchMap` | `runLatest(fn)` |
| `ignoreWhileBusy` | `exhaustMap` | `runUnlessBusy(fn)` |
| `recoverAsAction` | `catchError` | `onError(err => of(action))` |
| `startWithInitial` | `startWith` | `beginWith(initial)` |

The suffix grammar of `rxjs-operator-renaming` maps token for token: `Time` is `Every`, `Count` is `Of`, `On` is `On`, `When` is `When`, `Until` is `Until`, `Toggle` is `Between`, `While` is `While`, `By` is `By`, `With` is `With`, `Map` is `run*`, `All` is `flatten*`, `Scan` is `accumulate*`, `OnComplete` is rule 4 and `Final`. Its four roots `merge`, `concat`, `switch`, `exhaust` are the four concurrency suffixes here.

### Decisions

1. `On` for a repeating signal, `When` for a function, `Until` for a terminal signal. This matches `rxjs-operator-renaming` and the RxJS names `bufferWhen`, `windowWhen`, `delayWhen`. Rejected: one `When` overloaded by argument type, because the name would no longer show which shape it takes.
2. `accumulate` for `scan`. `running` was one letter away from `run`, and `mergeScan` would have been `runningConcurrent` next to `runConcurrent`.
3. `forkJoin` stays as `finalOfEach(sources)`, the only creation function in the list. Other creation functions stay out of scope.
4. Aggregates carry no suffix, rule 4. The completion dependency lives in the root meaning, not in an `OnComplete` suffix.
5. `then` and `as` stay short.
6. `sharedLinger`, `sharedInside`, `thenEvenIfFailed`, `finalOfEach` are new words for rare operators. Better words welcome.

### Renamed from the first draft

The first draft used these names. Applying the grammar renamed them.

| First draft | Now | Why |
| --- | --- | --- |
| `firstWhen(signal$)` | `firstWhen(fn)` | `throttle` takes a per-value function, not a signal |
| `lastWhen(signal$)` | `lastWhen(fn)` | `audit` takes a per-value function |
| `pollWhen(signal$)` | `pollOn(signal$)` | one repeating signal is `On` |
| `batchWhen(signal$)` | `batchOn(signal$)` | same |
| `spanWhen(signal$)` | `spanOn(signal$)` | same |
| `batchUntil(fn)` | `batchWhen(fn)` | a fresh closer per batch is `When`; `Until` ends the stream |
| `spanUntil(fn)` | `spanWhen(fn)` | same |
| `lastOnFrame()` | `lastEveryFrame()` | the frame is the clock; `On` is a signal |
| `laterBy(ms)` | `later(ms)` | `By` is a key selector |
| `failAfter(ms)` | `failIfQuiet(ms)` | `After` means wait; the missing value is `Quiet`; pairs with `failIfEmpty` |
| `sharedUntilQuiet(ms)` | `sharedLinger(ms)` | no subscribers is not `Quiet`; a duration is not `Until` |
| `runAll(fn)` | `runConcurrent(fn)` | the policy word; `All` meant two things |
| `again(fn)` | `runRecursive(fn)` | `expand` is `run` with feedback; `again` sounds like `retry` |
| `running(fn, seed)` | `accumulate(fn, seed)` | decision 2 |
| `onlyLast()` | `onlyFinal()` | `Last` is the latest so far; `Final` is at completion |
| `dropValues()` | `onlyEnd()` | joins `only*`; `drop` was a one-off root |
| `latestOf(other$)` | `combinedWith(other$)` | `Of` is a count; joins `*With` |
| `whenAllDone(others)` | `finalOfEach(sources)` | `When` is a token; decision 3 |
| `emitOn(scheduler)` | `observeOn(scheduler)` | kept technical, like `subscribeOn` |
| `notices()` | `asNotices()` | `as` is the replace root |
| `valuesFromNotices()` | `fromNotices()` | the inverse of `asNotices` |

`batchBetween` and `spanBetween` now spell their closing argument `fn`, as the Arguments table does. The `connected*` rows name `connectable` recipes instead of the deprecated `publish*`.

The grammar also added 19 names, 21 rows, for operators the first draft had not covered: `firstAndLastWhen`, `ifQuiet`, `sharedInside`, `flattenConcurrent`, `flattenLatest`, `flattenInOrder`, `flattenUnlessBusy`, `flattenCombined`, `flattenPaired`, `firstMatchIndex`, `allMatch`, `skipSeenBy`, `sameAs`, `isEmpty`, `accumulateConcurrent`, `accumulateLatest`, `retryAfter`, `repeatAfter`, `thenEvenIfFailed`. They are in the Names table.

Excluded, deprecated in RxJS 7 and gone in 8: `pluck`, `mapTo`, `concatMapTo`, `mergeMapTo`, `switchMapTo`, `exhaust`, `flatMap`, `combineAll`, `publish`, `publishBehavior`, `publishLast`, `publishReplay`, `multicast`, `refCount`, `retryWhen`, `repeatWhen`, `timeoutWith`, and the operator forms of `combineLatest`, `concat`, `merge`, `race`, `zip`, `partition`, `onErrorResumeNext`. The `connected*` names describe `connectable()` recipes, not `publish*`.

Deliberately unnamed: trailing-only `throttleTime`. It is close to `lastEvery` but not the same, and a name that hides the difference is worse than the original.

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

`later` (`delay`) is neither. It shifts every value and drops nothing.

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

`batchOn(signal$)` closes when the signal emits, then starts again. `buffer`.

```text
source     a--b--c--d--e--|
signal     ------x--------x
batchOn    ------[a,b]----[c,d,e]|
```

`batchWhen(fn)` asks for a new closing signal each time a batch opens. `bufferWhen`.

```text
source      a--b--c-----d--e--|
close       ------x-----------x
batchWhen   ------[a,b]-------[c,d,e]|
```

`batchBetween(open$, fn)` opens on `open$` and closes with the signal `fn` makes for that opening. `bufferToggle`. Overlaps stay overlaps.

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

`spanOn(signal$)` closes the current observable when the signal emits. `window`.

```text
source    a--b--c--d--e--|
signal    ------x--------x
spanOn    +-----+--------+
          a--b| c--d--e|
```

`spanWhen(fn)` asks for a fresh closing signal per span. `windowWhen`.

```text
source     a--b--c-----d--e--|
spanWhen   +-----+-----------+
           a--b| c-----d--e|
```

`spanBetween(open$, fn)` opens an observable on `open$`. `windowToggle`.

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
| `firstWhen(fn)` | `throttle` | Emit the opener, then ignore until the signal `fn` makes for it fires. |
| `firstAndLastWhen(fn)` | `throttle`, both edges | Emit the opener, and the last value if a later one arrived before the signal fires. |
| `lastEvery(ms)` | `auditTime` | Hold values for `ms`, then emit the latest. |
| `lastWhen(fn)` | `audit` | Hold values until the signal `fn` makes fires, then emit the latest. |
| `lastEveryFrame()` | `auditTime(0, animationFrameScheduler)` | The next paint is the window. Emit the latest value before it. |
| `pollEvery(ms)` | `sampleTime` | On a fixed clock, emit the latest value if one arrived. |
| `pollOn(signal$)` | `sample` | When `signal$` emits, emit the latest value if one arrived. |
| `afterQuiet(ms)` | `debounceTime` | Emit the latest value only after `ms` with nothing new. |
| `afterQuietWhen(fn)` | `debounce` | Each value picks the signal that ends its quiet wait. |
| `later(ms)` | `delay` | Shift each value by `ms`, or until a date. Drop nothing. |
| `laterWhen(fn)` | `delayWhen` | Each value waits for its own signal. |
| `failIfQuiet(ms)` | `timeout` | Error if no value arrives within `ms`. |
| `ifQuiet(ms, other$)` | `timeout({ each: ms, with: () => other$ })` | Switch to `other$` if no value arrives within `ms`. |
| `withGap()` | `timeInterval` | Add the time since the previous value. |
| `withTime()` | `timestamp` | Add the time the value was emitted. |
| `batchEvery(ms)` | `bufferTime` | Emit an array of every value in the duration. |
| `batchOf(n)` | `bufferCount` | Emit an array of every `n` values. |
| `batchOn(signal$)` | `buffer` | Close the array when `signal$` emits, then start another. |
| `batchWhen(fn)` | `bufferWhen` | Each batch asks `fn` for the signal that closes it. |
| `batchBetween(open$, fn)` | `bufferToggle` | Open an array on `open$`, close it with the signal `fn` makes for that opening. |
| `spanEvery(ms)` | `windowTime` | Emit an observable for each duration. |
| `spanOf(n)` | `windowCount` | Emit an observable for every `n` values. |
| `spanOn(signal$)` | `window` | Close the observable when `signal$` emits, then start another. |
| `spanWhen(fn)` | `windowWhen` | Each span asks `fn` for the signal that closes it. |
| `spanBetween(open$, fn)` | `windowToggle` | Open an observable on `open$`, close it with the signal `fn` makes for that opening. |
| `shared()` | `share` | One upstream while anyone listens. Restart after idle. |
| `sharedLast()` | `shareReplay({ bufferSize: 1, refCount: true })` | Late subscribers get the latest value, then live values. |
| `sharedRecent(n)` | `shareReplay({ bufferSize: n, refCount: true })` | Late subscribers get the last `n` values. |
| `sharedRecent(n, ms)` | `shareReplay` with `windowTime` | Replay buffer also drops values older than `ms`. |
| `sharedFinal()` | `share` with `AsyncSubject` | Emit only the last value, and only on complete. |
| `sharedLinger(ms)` | `share` with a reset timer | Keep the upstream for `ms` after the last subscriber leaves. |
| `sharedInside(fn)` | `connect` | `fn` builds the pipeline that uses the shared source. Connects on subscribe. |
| `connected()` | `connectable` | Subscribers wait until `connect()`. |
| `connectedLast(seed)` | `connectable` + `BehaviorSubject` | Manual connect, late subscribers get the seed or latest value. |
| `connectedRecent(n)` | `connectable` + `ReplaySubject` | Manual connect, replay the last `n`. |
| `connectedRecent(n, ms)` | `connectable` + `ReplaySubject` with `windowTime` | Manual connect, replay aged by `ms`. |
| `connectedFinal()` | `connectable` + `AsyncSubject` | Manual connect, emit the last value on complete. |
| `runConcurrent(fn)` | `mergeMap` | Start every inner. Emit as they arrive. |
| `runLatest(fn)` | `switchMap` | Unsubscribe the previous inner. Keep the newest. |
| `runInOrder(fn)` | `concatMap` | Queue inners. Start the next when the current completes. |
| `runUnlessBusy(fn)` | `exhaustMap` | Drop new source values while an inner is active. |
| `runRecursive(fn)` | `expand` | Feed each output back through `fn`. |
| `flattenConcurrent()` | `mergeAll` | The source emits streams. Subscribe to every one. Emit as they arrive. |
| `flattenLatest()` | `switchAll` | The source emits streams. Unsubscribe the previous one. Keep the newest. |
| `flattenInOrder()` | `concatAll` | The source emits streams. Queue them. Start the next when the current completes. |
| `flattenUnlessBusy()` | `exhaustAll` | The source emits streams. Drop new ones while one is active. |
| `flattenCombined()` | `combineLatestAll` | The source emits streams. Once all have emitted, emit the latest of each on every change. |
| `flattenPaired()` | `zipAll` | The source emits streams. Pair their values by index. |
| `as(fn)` | `map` | Replace each value. |
| `keep(pred)` | `filter` | Pass values that match. |
| `peek(fn)` | `tap` | Look at a value without changing the stream. |
| `onEnd(fn)` | `finalize` | Run when the subscription ends, for any reason. |
| `onlyFirst()` | `first` | Emit one value, then complete. |
| `onlyFinal()` | `last` | Emit the last value on complete. |
| `exactlyOne()` | `single` | Error unless the source emits exactly one match. |
| `at(index)` | `elementAt` | Emit the value at that index, or error. |
| `firstMatch(pred)` | `find` | Emit the first value that matches, then complete. |
| `firstMatchIndex(pred)` | `findIndex` | Emit the index of the first value that matches, then complete. |
| `allMatch(pred)` | `every` | Emit `true` on complete if every value matched, `false` as soon as one did not. |
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
| `skipSeenBy(key)` | `distinct(key)` | Drop values whose key was already seen. |
| `sameAs(other$)` | `sequenceEqual` | Emit `true` on complete if both sources emitted the same values in the same order. |
| `beginWith(...values)` | `startWith` | Emit these values first. |
| `finishWith(...values)` | `endWith` | Emit these values on complete. |
| `ifEmpty(value)` | `defaultIfEmpty` | Emit `value` if the source completed empty. |
| `failIfEmpty()` | `throwIfEmpty` | Error if the source completed empty. |
| `isEmpty()` | `isEmpty` | Emit `true` on complete if no value arrived, `false` as soon as one does. |
| `onlyEnd()` | `ignoreElements` | Emit nothing. Pass complete and error. |
| `withPrevious()` | `pairwise` | Emit `[previous, current]`. |
| `accumulate(fn, seed)` | `scan` | Emit each step of a fold. |
| `accumulateConcurrent(fn, seed)` | `mergeScan` | Each step is inner work. Start every one. Emit each result as the new state. |
| `accumulateLatest(fn, seed)` | `switchScan` | Each step is inner work. Unsubscribe the previous one. Keep the newest. |
| `fold(fn, seed)` | `reduce` | Emit the fold once, on complete. |
| `collect()` | `toArray` | Emit every value as one array on complete. |
| `count()` | `count` | Emit how many values passed. |
| `least()` | `min` | Emit the smallest value on complete. |
| `greatest()` | `max` | Emit the largest value on complete. |
| `onError(fn)` | `catchError` | Replace the error with another stream. |
| `retry(n)` | `retry` | Resubscribe after an error. |
| `retryAfter(ms)` | `retry({ delay: ms })` | Resubscribe `ms` after an error. |
| `retryAfter(fn)` | `retry({ delay: fn })` | Each error picks the signal that allows the resubscribe. |
| `repeat(n)` | `repeat` | Resubscribe after complete. |
| `repeatAfter(ms)` | `repeat({ delay: ms })` | Resubscribe `ms` after complete. |
| `repeatAfter(fn)` | `repeat({ delay: fn })` | Each completion picks the signal that allows the resubscribe. |
| `groupBy(key)` | `groupBy` | Split into one observable per key. |
| `withLatest(other$)` | `withLatestFrom` | Pair each value with the latest from `other$`. |
| `pairedWith(other$)` | `zipWith` | Pair values by index. Wait for both. |
| `combinedWith(other$)` | `combineLatestWith` | Emit when either side emits, with the latest of each. |
| `then(other$)` | `concatWith` | Subscribe to `other$` after this source completes. |
| `thenEvenIfFailed(other$)` | `onErrorResumeNextWith` | Subscribe to `other$` after this source completes or errors. |
| `alongWith(other$)` | `mergeWith` | Merge this source with `other$`. |
| `firstToEmit(other$)` | `raceWith` | Keep whichever source emits first. |
| `finalOfEach(sources)` | `forkJoin` | Emit the last value of each source when all complete. |
| `observeOn(scheduler)` | `observeOn` | Deliver notifications on that scheduler. |
| `subscribeOn(scheduler)` | `subscribeOn` | Call subscribe on that scheduler. |
| `asNotices()` | `materialize` | Turn next, error, and complete into notice values. |
| `fromNotices()` | `dematerialize` | Turn notices back into notifications. |

## Contributors

The main contributor to this project is SuperGrok (`supergrok@x.ai`).
