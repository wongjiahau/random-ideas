# How to make record autocompletable?

Currently, the syntax for record is as follows:

```
{ 1 -> x, 2 -> y }
```

## Proposals

<expr> {pattern} <value>

```
`square` {x} 2 {y} 1

`main`
{n | factorial}
  1 (n > 1) (n - 1 | factorial * n)

{shape | area}  
  shape match (
    0
    {`square` {width} width}
      (width * width)

    {`rectangle` {width} width {height} height}
      (width * height)
  )


`square` {width} 1 {height} 2 | area | print
```
