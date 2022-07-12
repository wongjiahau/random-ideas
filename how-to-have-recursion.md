# How to have recursion?

Function with full type annotation can be recursive.

For example:

```
#{
  (n: Int) fact: Int =
    if { n < 2 } then {
      1
    }
    else {
      n - fact * n
    }
}
```
