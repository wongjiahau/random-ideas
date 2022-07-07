# Harmonize let, object, and suspended expression, lambda, pattern matching

## Proposal 1

Use `{}` for closure, `#{}` for object, `{}()` for let-expression

Grammar:

```
closure
= ‘{’ body ‘}’

function
= ‘{’ branch+ ‘}’

body
= (entry ‘,’)\* expr

branch
= ‘|’ pattern (’|’ pattern)\* ‘->’ body

object
= ‘#{’ (entry (’,’ entry)\*)? ‘}’

entry
= assignment
| expr

assignment
= pattern type_annot? ‘=’ expr
| function_decl
```

## Problem 1.1

Using let-expression is counterintuitive due to the need to call the closure.

## Solution 1.1

Special syntax for function declaration, where ‘=’ can be omitted.

```
(a: Bool) and (b: Bool): Bool {
  (a, b) |> {
  | (true, true) -> true
  | _ -> false
  }
}
```

## Problem 1.2

The function syntax `'{' branch+ '}'` does not allow block expression for each
branch, for example:

```
(shape: Shape) area: Float {
  shape |> {
    Square #{side} -> // how to have block expression here?
  }
}
```

## Solution 1.2(a)

Change the function syntax to:

```
function
  = ('|' branch)+
  | block
branch
  = pattern '->' body

body
  = expr
  | block
```

For example:

```
(shape: Shape) area: Float {
  shape |> (
  | Square #{side} -> {
      x = side + 1,
      y = side - 1,
      x + y
    }
  | Circle #{radius} ->
      radius square * PI
  )
}

area =
  | Square #{side} -> side ^ 2
  | Circle #{radius} -> radius ^ 2 * PI

```

However, this creates weird indentatioin.

# Other Notes

Top-level syntax is object without need to declare #{}.

Also, `-` means private.
