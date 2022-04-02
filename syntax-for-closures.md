# Syntax for Closures

## Problem

Currently, the syntax for closures is the usual lambda, for example: `a -> b`.

This syntax is ok, however it is a bit tedious when user wants to create
suspended operation, say that want to write their own control flow function.

Let's call this function `else`:

```
default else branches:
  branches
    List.findMap (cond `then` value -> cond call () match {
      true: value call () `some` (),
      false:  `none`
    })
    match {
      value `some` (): value,
      `none`: default call ()
    }

`F`
  unless [
    (() -> score > 80) `then` (() -> `A`),
    (() -> score > 60) `then` (() -> `B`),
  ]
```

Notice how the usage of `else` is cumbersome because of `() ->`

## Proposal

We can mimic the closure syntax of Ruby which is just `{}`.

Then the syntax for lambda is `{(params ',')* body}`.

The above example can be re-written as such:

```
default else branches:
  branches
    List.findMap { cond `then` value,
      cond call () match (
        true: value call () `some` (),
        false:  `none`
      )
    }
    match (
      value `some` (): value,
      `none`: default call ()
    )

`F`
  unless [
    { score > 80 } `then` { `A` },
    { score > 60 } `then` { `B` },
  ]
```
