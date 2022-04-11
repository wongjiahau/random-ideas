# Flexible Tagged Union

## Motivation

To allow user to create even friendlier DSL,
we should allow named typed to be used to construct union
(sort of like GraphQL union).

For example, if we use only variants for construct in a simple math expression:

```
(expr: (1 | 'number') '+' (2 | 'number'))
```

It can get a little verbose due to the `'number'` wrapper.

So, if the based number typed can be discriminated at runtime, then we can
rewrite the code above less verbosely:

```
(expr: 1 '+' 2)
```

## Proposal

To pattern match on named types, we can use the `::` syntax.

For example, to evaluate the math expression above:

```
(
  eval: expr match {
    n :: Number:
      n,

    a '+' b:
      (a | eval) + (b | eval)
  }
)
```
