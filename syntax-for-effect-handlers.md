# Syntax for effect handlers

## Introduction

Effect handlers has 2 components:

- perform
- handler

Perform contains a name and a payload.

Handler contains a name, and two methods:

- return (transform the type of body to another type, ususally for wrapping)
- handle (a function that has the continuation parameter)

## Proposal 1

```
main: {
  "hello"
    perform throw
    handle (
      throw: { content: content | resume },
      return: {x: x}
    )
}
```
