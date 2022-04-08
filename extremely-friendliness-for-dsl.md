# Extreme friendliness for DSL

## Purpose

This article explore syntax ideas that simplifies the creation of DSL.

## Proposal

In a language, we can have 4 types of identifiers:

- Single quoted string
- Double quoted string
- Backtick quoted string
- Non-quoted alphanumeric string

And in IBX there's 2 kinds of identifiers:

- Variables
- Variants

Since DSL utilize Variants a lot, to ease the creation/usage of DSL,
we should treat non-quoted alphanumeric string as identifiers.

However, this means that Variables must be one of the quoted string.
Which can be very cumbersome for writing code (especially the interpretations of DSL).

## Examples (single-quoted-string = variable, non-quoted alphanumeric string = variant)

DSL for simple Math expressions:

```js
(
  'eval': {
    'a' + 'b':
      'a' | 'eval' '+' ('b' | 'eval'),

    n of 'x':
      'x'
  },

  'main': {
    (n of 1) + (n of 2) + (n of 3) | 'eval'
  }
)

```

As seen above, the implementation part becomes too verbose due to all the single-quoted-strings.

## Proposal 2 (Special environment)

Let the default lexing mode be non-quoted strings as variable,
while single-quoted-string as variants.

However, when there's a special marker or surround, then the lexing rule will
be inverted (kind of).

For example (special wrapper is triple quote, to interpolate use `\()`):

```
(
  eval: {
    a '+' b:
      a | eval + (b | eval),

    'n' 'of' x:
      x
  },

  bigNum: 'n' 'of' 999,
  main: {
    '''(n of 1) + \(bigNum) + (n of 3)''' | eval
  }
)
```
