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
