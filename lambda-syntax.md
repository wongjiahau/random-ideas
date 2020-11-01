# Lambda Syntax
This document is compares the new syntax idea with Typescript syntax.

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
  = identifier [typeAnnotation] "=" expr

typeAnnotation 
  = ":" type

expr
  = expr' [typeAnnotation]
  
expr'
  = lambda
  | application
  
lambda 
  = "{"  {variable ";"}* expr "}"
  
variable 
  = identifier typeAnnotation [("=" | "?=") expr]
  
application
  = expr "." "(" {[identifier "="] expr ","}* ")"
```
## no argument 
```js
// js
const f = () => {console.log('hello')}
f()
// new
f = {console.log('hello')}
_.f
```
## with arguments
```js
// js
f = (x: number) => x + 1
f(1)
// new
f = {x: number; x.plus(1)}
1.f
```

## optional arguments
```ts
//ts
slice = (s: string, from, to = s.length - 1) => 
  s.slice(from, to)
// new
slice = {
  s;
} : string
```