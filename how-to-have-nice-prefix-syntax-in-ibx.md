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

## Proposal 3

Treat `'f` as a function that consumes 1 argument.  
Treat `"f` as a function that consumes 2 arguments.

For example,

```
("f a 'g c ) = ("f a ('g c))
```

## Proposal 4

Have an operator `o`, such that `f o x` means `f(x)`.
For example, say we represent `o` as `:`:

```
#{
  if: condition: body = condition: () {
    true -> body: (),
    false -> #{
      else: other = other: ()
    }
  },

  example: () =
    if: { 3 > 4 }: {
      `Fine`
    }
    else: {
      `Nope`
    }
}
```

Cons of this proposal is that `:` cannot be used for type annotations anymore.

## Proposal 5

Haskell-like syntax, where `f x` means `f(x)`. Then add-on `x.f` means `f(x)`.

Example:

```
#{
  if condition body = condition.{
    true -> body (),
    false -> {
      elif condition body = if condition body,
      else other = other ()
    }
  },

  example () =
    if { 3 > 4 } {
      `Fine`
    }
    .elif { xs.some { it > 2 } } {
      `Oh`
    }
    .else {
      `Ok`
    }
}
```

## Question

Is prefix syntax necessary?

```
yes
```
