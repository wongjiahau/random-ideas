# Adhoc Polymorphism might be necessary

Adhoc Polymorphism (aka name overloading) might be necessary because of
ergonomics and readability.

Consider the following code, written in OCaml style:

```

myName str.includes `Hello` bool.and (hisName str.endsWith `Ben`)
```

The explicit namespace resolution such as `str` and `bool` are noisy and thus
affects readability, in contrast to this:

```
myName includes `Hello` and (hisName endsWith `Ben`)
```

## Proposal 1 (OOP)

Classic OOP allows function (or methods) names to be overloaded by hiding each function under an object.

This does not requires typechecking to work, however there is no easy way to add new methods.

## Proposal 2 (Single Dispatch)

This was implemented in the previous version of KK, where the compiler can
determine which function is being called based on the type of the first argument.

However, this will require typechecking.
