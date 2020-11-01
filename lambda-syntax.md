# Lambda Syntax
This document is compares the new syntax idea with Javascript syntax.

## Grammar
Repitition rules:
```
{ x }* = 0 or more times of x
{ x }+ = 1 or more times of x
{ x }? = 0 or 1 time of x
[ x ] = { x }?
```
```bnf
lambda = "{" 
  {argument ","}*
"}"
argument = identifier 
```
## no argument 
```js
// js
() => {console.log('hello')}
// new
{console.log('hello')}
```
## with arguments
```js
// js
(x) => x + 1
// new
{x x.+(1)}
```