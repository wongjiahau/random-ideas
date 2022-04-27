# Tuple and Object

## Preface

Currently, IBX lacks the syntax for tuples, which is crucial for some DSL that
requires shortform, for example coordinates `(1, 2)`.

## Proposal

Implicitly assign numeric keys starting from 0 for values without keys in an object.

The numeric key only increments for values without keys.

For example (note that double-quoted strings are identifiers):

## Syntax A

```ts
(
  o: (8, 9),
  o."0" | print // 8,
  o."1" | print // 9,

  ("0", x, y): (x: 1, 8, y: 2),
  "0" | print // 8,
  x | print // 1,
)
```

## Problem A

This proposal has ambiguity, how do we know if `(x)` means `("0": x)` or `(x: x)`?

## Syntax B

Uses `:value` for key-indexed values.

```
(
  o: (: 8, : 9),
  o."0" | print // 8,
  o."1" | print // 9,

  (:a, x, y): (x: 1, : 8, y: 2),
  a | print // 8,
  x | print // 1,
)
```

## Problem B

This syntax is hard for creating DSL that need tuples.

## Syntax C

Use `key:` for record punning.

```
(
  o: (8, 9),
  o."0" | print // 8,
  o."1" | print // 9,

  (a, x:, y:): (x: 1, 8, y: 2),
  a | print // 8,
  x | print // 1,
)
```

This is the best so far as it allows DSL to be created easily, yet no ambiguity.
