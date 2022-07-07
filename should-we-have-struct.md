# Should we have struct?

Example of `struct`:

```
struct Hello (Int),

struct Shape #{
  side: Float
}
```

## Counterpoints

- Struct can already be emulated using a one-choice tagged union
- Struct syntax does not allow private constructor (not really, we can
  just add the private modifier)

## Good points

- Struct is much shorter than one-choice tagged union.
