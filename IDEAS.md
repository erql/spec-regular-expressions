# Ideas

This file contains different (sometimes not well-thought-through) ideas on the spec:

## Silent emission / Negative lookaround

`_` symbol will indicate that this is a negative capturing -- emission is required, but will be ignored in the output

`_AB_C` 

## Simultaneous emissions

A and B simultaneously

`[AB]`

## Take one A unless B emits

`[A!B]`

## Take A untill B emits

`[A*!B]`

## No-emissions

`A !B C`

or

`A ^B C`

## Emissions from A w/o interruption from B

`[A*^B]`

## Ensure time gaps

ensure timed gaps between emissions

`A 5ms A`

or

`A (^A 5ms) A`
`A (!A 5ms) A`

## non-capturing groups

`A (? B ) A`

## pipes in the expression

`A |> ${ filter(x => x) }`

## Names groups

`(:id A)`

Named groups emit Observables with `id` property