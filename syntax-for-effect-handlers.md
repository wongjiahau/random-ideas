# Syntax for effect handlers

## Introduction

Effect handlers has 2 components:

- perform
- handler

Perform contains a name and a payload.

Handler contains a name, and two methods:

- return (transform the type of body to another type, ususally for wrapping)
- handle (a function that has the continuation parameter)

## Question

1. Is `return` necessary? It's just like a syntax sugar for avoiding wrappers
2. Should Ibx support effects with multiple operation?

## Proposal 1

```
(
  main: {
    "hello"
      perform throw
      handle (
        throw: { content: content | resume },
        return: {x: x}
      )
  }
)
```

## Proposal 2

```
(
  main: {
    "hello"
      perform throw
      (handle | throw) {
        handle: { content: content | resume },
        return: { x: x }
      }
  }
)
```
