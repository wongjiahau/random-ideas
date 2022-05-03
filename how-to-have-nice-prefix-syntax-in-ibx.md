# How to have nice prefix syntax in IBX?

## Problem

Currently, the prefix function call in IBX is a little mouthful:

For example: `f <| x` as opposed to `f(x)`

And this applies to variants with single argument.

For example: `'Ok' <| 1` as opposed to `Ok(1)`

## Proposal

Using backslash, for example: `f \ x ` is the same as `x | f`.

## Question

Is prefix syntax necessary?
