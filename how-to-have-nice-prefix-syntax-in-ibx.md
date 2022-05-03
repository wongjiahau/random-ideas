# How to have nice prefix syntax in IBX?

## Problem

Currently, the prefix function call in IBX is a little mouthful:

For example: `f <| x` as opposed to `f(x)`

And this applies to variants with single argument.

For example: `'Ok' <| 1` as opposed to `Ok(1)`

## Why prefix is necessary?

Prefix syntax is necessary for constructing some DSL, for example HTML.

## Proposal 1

Using backslash, for example: `f \ x ` is the same as `x | f`.

## Proposal 2

Treat `a b` as `a(b)`, and `a b c d` as `a(c(b,d))`, and so on and so forth.

## Question

Is prefix syntax necessary?
