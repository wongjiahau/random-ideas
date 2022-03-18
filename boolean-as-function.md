# Boolean as function

## Definition:

```
a true b = b
a false b = a
```

## Question:

Does this works well for left-associative syntax?
Because it forces the condition list to be reversed.

## Example 1 (a true b = b):

```
score grade () =
  `F`
    (score > 40) `C`
    (score > 60) `B`
    (score > 80) `A`
```

## Example 2 (a true b = a):

```
score grade () =
  `A` (score > 80)
  `B` (score > 60)
  `C` (score > 40)
  `F`
```

The example above will look weird if formatted (prettier-esque):

```
score grade () =
  `A`
    (score > 80) `B`
    (score > 60) `C`
    (score > 40) `F`
```

## Evaluation strategy

1. Evaluate function first before evaluating arguments

## Implications

This concept can be generalized to any construct that requires 3 components, for example let-expression.
