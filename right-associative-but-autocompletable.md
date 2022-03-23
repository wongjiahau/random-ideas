# Right associative but autocompletable

## Question

Suppose in a language where all infix expressions are right-associative,
how do we still allow the common left-associative chaining?

## Proposal 1 (dot syntax)

Make dot the only left-associative operator, and has the dotted-expression has
higher precendence than normal infix expressions.

Examples:

```
// Without using dot syntax

print |
  (x -> x + 2) map
    (x -> x > 3) filter
      [1, 2, 3]

// With dot syntax
[1, 2, 3]
  .(map | x -> x + 2)
  .(filter | x -> x > 3)
  .(print)
```
