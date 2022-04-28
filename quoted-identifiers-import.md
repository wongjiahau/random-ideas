# Quoted Identifiers Import

Since IBX supports quoted identifiers, we can treat quoted identifiers that resembles URL as import.

So instead of writing:

```
(x: "./somefile.ibx" | import)
```

Just write:

```
(x: "./somefile.ibx")
```
