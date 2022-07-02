# Named Lambda

This is sort of like Javascript functions that can be named, for example:

```js
(function factorial(n) {
  if (n < 2) {
    return 1;
  } else {
    return factorial(n - 1) * n;
  }
})(10);
```

## Motivation

This is to simplify creation of tail recursive function.

## Proposal syntax 1

```
{
  factorial 0 = 1,
  factorial 1 = 1,
  factorial n = factorial (n - 1) * n
} 10
```

This means that anonymouse function has the syntax of:

```
{ _ arg = body }
```

For example:

```
xs . map { _ x = x  + 2}
```

Another example:

```
xs . map {
  length Nil = 0,
  length (Cons x xs) = length xs + 1
}
```

### Example of tail recurisve factorial

```
#{
  factorial n = {
    go 1 result = result,
    go n result = go (n - 1) (n * result)
  } n 1
}
```
