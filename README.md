# Regular Expressions for Reactive Streams 🐇

This repository defines specification of regular expressions for regular streams.

List of implementations:

- RxJS — rxjs-rexprs

## Intro

Let's imagine we're implementing a mouse drag-n-drop functionality.

We'll take three streams: `mousedown$`, `mouseup$` and `mousemove$`. These stream we will now combine to react to dragging action.

On mouse down we start listening to mouse moves. And continue listening to that until mouse up event.

Here is a marble diagram of the events queue:

```
mousedown$    --o----------------
mousemove$    ----o-o-o-o-o-o----
mouseup$      ----------------o--
```

One event from `mousedown$`, multiple events from `mousemove$` and we finish with one event from `mouseup$`.

Let's update our diagram to mark the events according to their respective stream:

```
mousedown$    --d----------------
mousemove$    ----m-m-m-m-m-m----
mouseup$      ----------------u--
```

Now we can merge this diagram into a single-line represetation:

```
dnd$          --d-m-m-m-m-m-m-u--
```

And one last transformation: let's drop the marble notation and just write it down as a string:

```
                d m m m m m m u
```

Or

```
                   dmmmmmmu
```

Now that its a simple string -- we could apply Regular Expressions to it!

```js
/d(m*)u/.exec('dmmmmmmu')

// ->

[ 'dmmmmmmu'
, 'mmmmmm'
]
```

We select all the `m` letters that are located between `d` and `u`!

And when we apply this concept to streams:

```
mousedown$    --d----------------
mousemove$    ----m-m-m-m-m-m----
mouseup$      ----------------u--

expression           d(m*)u

result$       ----m-m-m-m-m-m-|
```

We select all the `m` events that occured after `d` and before `m`!

## Spec

### Definitions

#### Expression

A string, representing a sequence of events

#### Capturing group

`()` defined in a pair of parenthesis, wrapping an expression

#### Stream

`ABC` capital letters represent streams

#### Repeat

Repeat `*` symbol indicates that prepending expression can occur `0` to `Infinity` times

#### Root expression

Whole expression

#### Root observable

Observable, emitted when the root expression is applied. It behaves as capturing group observable

#### Capturing group observable

Observable, emitted for a specific capturing group

### Rules

Once the root expression is applied, a Root observable is emitted.

Root observable emits a capturing group observable for each capturing group in the root expression.

If an expression has started matching events and then fails against the expression -- its observable emits an error and completes.

### Examples

**Single click:**

```
click$        --c---c----c-------

regex                 c 

result        --(c|)
```

**All clicks**

```
click$        --c---c----c-------

regex                 c*

result        --c---c----c-------
```

**All clicks as observables**

```
click$        --c---c----c-------

regex                (c)*

result        --c|
                 --c|
                    -----c|
                          -------
```


**DnD:**

```
mousedown$    --d----------------
mousemove$    ----m-m-m-m-m-m----
mouseup$      ----------------u--

regex                dm*u

result        ----m-m-m-m-m-m-|
```

## Ideas:

check out IDEAS.md

## Questions and Answers:

Q:

Given an expression:

`mousedown$ mousemove$* mouseup$`

how following situations should be handled:

- up$ never emits

A: the resulting observable never completes

- down$ emits once again before up$

A: it is ignored

TODO:
    Maybe, with introduction of negative queries (something like `D[M!D]*U` -- mousemoves, but `!` NOT mousedowns) the match search should be restarted, and current Observable should error
    Maybe, matching should restart with this new emission. This behaviour could be explicitly defined via some syntax addition.

- down$ emits once again after up$

A: the observable will complete with up$ event, expression needs to be wrapped into a group with repeat `( ... )*` to listen to another cycle of down-move-up events

- down$ or up$ completes or errors

A: output errors

---