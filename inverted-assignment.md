# Inverted assignment

To ease parsing, assignment can be made left associative,
and to reduce the usage of parentheses, we can invert assignment.

This note is to explore the usability of this idea.

## Example

Note that we can apply the same logic for lambda (i.e. inverted).

```c
{
  // Storing a value into a variable
  12 + 24
    -> x,

  // defining square function
  n * n
    <- n
    -> square,

  // ignoring the result of an expression
  12 | square | print
    -> _,

  // factorial
  0 (n < 2) (n - 1 | factorial * n)
    <- n
    -> factorial

  // pattern matching
  {
    {
      side * side
        when ({side} | `square`),

      radius | square * PI
        when ({radius} | `circle`)
    }
     -> area
  }
    -> shape,

  {12 -> side} | `circle` | shape.area | print
    -> _
}
```

## What if we right-associative everything?

```
{
  // Storing a value into a variable
  x = 12 + 24,

  // defining square function
  square = n -> n * n,

  _ = print | square | 12,

  // factorial
  factorial = n -> 0 (n < 2) (n * factorial | n - 1),

  // pattern matching
  shape = {
    area = {
      ({side} | `square`) -> side * side,
      ({radius} | `circle`) -> PI * square | radius,
    }
  }

  _ = print | shape.area | `square` | {side = 12}
}
```

Right-associative code actually looks much better and more conventional,
however there's one problem which is there's no natural syntax for chaining
functions like this:

```js
// Javascript
[].map((x) => x * x).filter((x) => x > 2);
```

In a right-associative world, the above code would have been written like this:

```c
(x => x > 2)
  filter (x => x * x)
  map []

// Or
([] map x => x * x) filter x => x > 2
```

_But is function chaining really important?_
