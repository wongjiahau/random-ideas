# Let expression and object

## Preface

Currently, there are no way to express let expression, the only way is to
do this:

```
(
  result = (
    x = 1,
    y = 3,
    return = x + y
  ).return
)
```

This is very verbose and not ergonomic.

## Proposal 1

The last object element (key-value pair OR expression) should be returned.

For example:

```
(
  result = (
    x = 1,
    y = 2,
    x + y
  )
)
```

Then how to return an object?
We can assign a special token that represents "this".

Say we use the symbol `$`.

Then to create an object:

```
(
  myCar = (
    name = "Proton",
    age = 99,
    $
  ),

  coordinate = (0, 1, $)
)
```
