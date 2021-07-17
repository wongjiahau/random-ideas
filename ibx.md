# IBX (Infix Binary Expression)

## Reserved symbols
| Symbol | Usage | Example | 
| -- | -- | -- | 
| `'` | Construct opening ternary operator. |  `'f`  |
| `'` | Construct closing ternary operator. |  `g'`  |
| `''` | Declare ternary operator. | `f''g`
| `(` `)` | Grouping precendence. | `(x f y)` |

## Transformation
|Rule|IBX| S-Expression | Note | 
| --|--| -- |  -- |
| Binary operator | `x f y` | `(f x y)` | -- |
| Ternary operator | `x f' y 'g z` | `(f''g x y z)` | `f''g` is a single identifier | 
| Left associative | `x f y g z` | `(g (f x y) z)`| | 
| Ternary center higher precendence | `x f' a b c 'g z` | `(f''g x (b a c) z)` | Treat `f'` as opening bracket `(`, and `'g` as closing bracket `)` |
| Ternary operator behaves like binary operator | `x f' y 'g z j k` | `(j (f''g x y z) k)` | Treat `f' ... 'g` as a binary operator |  

## Description
IBX is like S-expression, where it does not only cover value-level syntax, it can be also used for language-level syntax.  

For example in Lisp, variable declaration is also using S-expression:
```
(defvar x 234)
```
Similarly, in IBX:
```
x = 234
```


## Example
```js
(2 <' 3 '< 4 |> print)

def' factorial '= (
  0 
  unless' n > 2 'then 
    (n - 1 |> factorial * n)
  <- n
)

def' <''< '= 
  a < b & (b < c) <- c <- b <- a
  : (bool <- int <- int <- int)
```
Notice that lambda is define in reversed, so `a <-
b` means `b -> a`, this is because IBX is left
associative.
