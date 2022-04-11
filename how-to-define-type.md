# How to define types?

## Problem

Currently there's no way to define types in Ibx.
We need to have two kind of type declaration:

- Alias type
- Named type

## Why is named type needed?

Primarily:

- For easing implementation of recursive types (e.g. Cons List)
- Less-verbose error messages (counter-example: Typescript)
- Allow user to implement encapsulation (private type, public constructor function)
- Faster auto-complete suggestion if user used named type

## Question

1. Should type share the same namespace as values?
   If they share the same namespace, then we cannot have the type named `List`
   and module name `List` at the same time.

## Proposal

Type can be scoped.
