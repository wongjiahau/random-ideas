# Multiple precedence

## Motivation

To enable user to create even more natural DSL, IBX should support multiple
precedence, two aspects: high/low and right-associative/left-associative.

## Prior work

Language like Haskell and Agda allow user to define custom operators with a set
precedence.

However, there is an issue with this, as the user using these operators will
not know their precedence without looking up the definition.

This makes parsing a piece of code uneasy as user has to switch back and forth
between the code and the precedence definition.

## Proposal 1

Assuming identifiers can be prefixed or postfixed with multiple $symbol.

More single $symbol means lower precedence, prefixed single $symbol means
left-associative, postfixed single $symbol means right-associative.

For example, let us try reproducing the SQL Conditional precedence, when `and`
has lower precedence than `or` and `or` has lower precedence than mathematical
expression.

Say $symbol is `.`:

```
3 + 2 .> 5 ..or something ...and 1 + 2 ..or 3
```

Will be parsed as:

```
(((3 + 2) > 5) or something) and ((1 + 2) or 3)
```

Another example, right-associative cons list:

```
1 &. 2 &. 3 &. 4 &. 'nil'
```

Will be parsed as:

```
1 &. (2 &. (3 &. (4 &. 'nil')))
```
