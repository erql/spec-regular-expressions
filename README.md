# Regular Expressions for Reactive Streams 🐇

This repository defines specification for reactive regular expressions.

List of implementations:

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

### Vocabulary

#### Expression

A string, representing a sequence of events

#### Capturing groups

`()` defined in a pair of parenthesis, wrapping an expression

#### Streams

`ABC` capital letters represent streams

#### Repeat

`*` symbol indicates that prepending expression can occur `0` to `Infinity` times

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

**Single click (exclusive):**

```
click$        --c---c----c-------

regex                 c 

result        --|
```

**Single click (captured):**

```
click$        --c---c----c-------

regex                (c)

result        --c|
```

**All clicks (captured)**

```
click$        --c---c----c-------

regex                (c*)

result        --c---c----c-------
```

_.alternative._

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

regex          d(m*)u

result        ----m-m-m-m-m-m-|
```


## Examples

DnD:

```js
tag`(${mousedown}${mousemove}*${mouseup})*`
```

Consequent clicks:

```js
tag`(${click}${click})*`
```

Consequent clicks w/o outside clicking:

```js
tag`(${click} ^${outsideClick} ${click})`
```

Timely clicks:

```js
tag`(${click} ${click})5ms`
```

# ===========

## Ideas and TBD:

no-Ideas and 

`A !B C`

or

`A ^B C`

ensure timed gaps between emissions

`A 5ms A`

or

`A (^A 5ms) A`
`A (!A 5ms) A`

non-capturing groups

`A (? B ) A`

pipes in the expression

`A |> ${ filter(x => x) }`

## Questions:

Q:

Given an expression:

`mousedown$ (mousemove$*) mouseup$`

how following situations should be handled:

- up$ never emits

A: the resulting observable never completes

- down$ emits once again before up$

A: ???

EITHER it should complete the previous emission
OR     it should ignore it
OR     root group should emit another subgroup

maybe it should be defined by some "greedy" operator

- down$ emits once again after up$

A: the observable will complete with up$ event, expression needs to be wrapped into `(` ... `)*` to listen to another cycle of down-move-up events

- down$ or up$ completes or errors

A: output errors

- down$ emits twice before up$

A: capturing group observable errors, completes and root observable emits again

---