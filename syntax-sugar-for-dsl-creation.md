# Syntax sugar for DSL Creation

```
(start: Int) to (end: Int) by (step: Int): Range = body
```

Should not be expanded as (to prevent ambiguity between function call and
member access):

```
to = {
  start -> {
    end -> #{
      by = {
        step ->
          body
      }
    }
  }
}
```

Instead, it should be expanded as such:

```
TempEnum1 = enum [ TempEnum1 #{start: Int, end: Int} ],
to
: Int -> (Int -> TempEnum1)
= {
  start -> {
    end -> TempEnum1 #{ start, end }
  }
},
by
: TempEnum1 -> (Int -> Range)
= {
  #{ start, end } -> {
    step ->
      body
  }
}
```
