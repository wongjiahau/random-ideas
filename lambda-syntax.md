# New language
This document is compares the New syntax with Typescript/Haskell's syntax.
Each syntax decision should be back with reasons, not just taste.

## Grammar
Repitition rules:
```
{ x }* = 0 or more times of x
{ x }+ = 1 or more times of x
{ x }? = 0 or 1 time of x
[ x ] = { x }?
```
```bnf
program 
  = {statement}+
  
statement 
  = declaration
  
declaration
  = valueDeclaration
  | typeDeclaration
  
valueDeclaration
 = [type] identifier "=" expr
 
 typeDeclaration
   = "type" identifier "=" type
 
type 
  = literalType
  | recordType
  | templateType
  | aliasType
  | functionType
  | sumTypes

expr
  = [type] expr'
  
expr'
  = lambda
  | application
  | record
  
lambda 
  = "{"  {assignable ";"}* expr "}"
  
assignable 
  = [type] assignable'

assignable'
  = 
  
application
  = expr "." "(" {[identifier "="] expr ","}* ")"
```
## Type annotation
Unlike most modern, New types are like C-like languages, where types comes before expressions. For example, instead of `x: number`, in New it's `number x`.
Reason:
1. allows better auto-complete experience
For example, you will get suggestions of `name` as soon as you reach the cursor.
```ts
type People = {string name; number age;}
p = People {n
            ^ assuming you typed until here
```
2. allow expressions to be typed in a more readable way. Especially with record type, it might look like as if the aliased record type is being use as a constructor, but instead it's just a type assertion.
For example, 
```ts
// In Elm
answer = {
  name = "John",
  kid = {
    hobby = "game"
  } : Kid
} : People

// In New
answer = People {
  name = "John",
  kid = Kid {
    hobby = "game"
  }
}
```
3. No new syntax is needed for specifying the return type of a function 
For example,
```ts
// In Typescript
const square = (x: number): number => x * x
                          // ^ special syntax

// Or using variable type annotation
const square = (x: number) => {
  const result: number = x * x
  return result
}

// In New, we just need to assert the returned expression to a specific type
square = {
  number x;
  number x.times(x)
  // ^ asserting the returned expression has number type
}

```
## Function
In New, function arguments are just variables that are not instantiated with any values.  
Reason: this allow New to have reduced grammars, which implies that:
1. parser is easier to write
2. more consistent syntax (better user experience)
3. refactoring function produce cleaner diff, for example:
```ts
// In typescript
const f = () => {
  const x: number = 5
  return sin(cos(x))
}
const g = () => {
  const x: number = 6
  return sin(cos(x))
}
// Refactor f and g into a common function h,
// which requires a lot of syntactical changes
// thus producing uglier diffs
const h = (x: number) => {
  return sin(cos(x))
}
const f = () => h(5)
const g = () => h(6)

// new
f = {
  number x = 5;
  x.cos.sin
}
g = {
  number x = 6;
  x.cos.sin
}
// refactoring does not change much of the original code
h = {
  number x;
  x.cos.sin
}
f = {5.h}
g = {6.h}
```
### No argument 
```js
// js
const f = () => {console.log('hello')}
f()
// new
f = {console.log('hello')}
_.f
```
### with arguments
```js
// js
f = (x: number): number => x + 1
f(1)
// new
f = {number x; number x.plus(1)}
1.f
```

### optional arguments
```ts
//ts
slice = (
  s: string, 
  from: number, 
  to: number = s.length - 1
): string => {...}
slice("Hello", 0)
slice("Hello", 0, 2)
// new
slice = {
  string s;
  number from;
  number to =? s.length;
  ...
}
"Hello".slice(0)
"Hello".slice(0, 2)
```

### keywords argument
```ts
// ts
f = ({a, b}: {a: number, b: number)) => {...}
f({a: 1, b: 2})
// new
f = { number a; number b; ...}
1.f(b=2)
```

### Function application
Function application (or invocation) in New uses the universal function call syntax (UFCS) because:
1. allows IDE to suggestion autocompletion easily
2. natural function chaining

