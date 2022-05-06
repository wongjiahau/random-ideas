# Custom Array Element Separator

## Preface

Currently, the only way to separate elements in an array literal is using comma.  
This can limit the flexibility of creating more accurate DSL.

## Proposal 1

Treat the first identifier after `[` as the separator. For example,
`[- 1 - 2 - 3]` is the same as `[1, 2, 3]`.

DSL example (conditional):

```
(
  branches else default:
    branches list.findMap {
      condition 'then' value:
        condition | call | {
          true: value | call | 'some',
          false: 'none'
        }
    } | {
      value | 'some': value,
      'none': default | call
    },

  [
    if { x > 80 } 'then' { 'A' }
    if { x > 60 } 'then' { 'B' }
  ]
    else { 'C' }
)
```

## Question

1. Should array with different separator have different type?

> Maybe, because the DSL author might want to enforce a consistent syntax.

2. Should we have another bracket for array that use comma as separator?

> Perhaps we can use `[| |]` and `[ ]`.

## Proposal 2

Let each element be separatable using multiple separators defined between `[` and `|`.

For example:

```
[if then|
  if { x > 80 } then { 'A' }
  if { x > 60 } then { 'B' }
]
```

`if` is used for separating elements within the array, `then` is used as separator within a tuple.

Also `if` and `then` will have the lowest precedence, except that they have higher precedence than brackets.

The above array has the type of `({ Boolean }, {['A', 'B']}) | List`.

## Question

It is very verbose for user to keep defining the separator tokens, perhaps the type system can determine it?
However, this means that typechecking has to be run before parsing at some
point, which can complicate the interpreter architecture.
