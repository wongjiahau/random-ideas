# Better Property Access Symbol

## Preface

When trying to emulate some code in the current syntax, I found that using `.`
as the property access operator can be hard to read.

For example:

```
myList list.map { .name } | list.length
```

I suspect this is because we are used to treating `.` as a sentence separator,
instead of a conjuction.

## Proposal

Use Rust `::` operator.

For example:

```
myList list::map { ::name } |> list::length
```
