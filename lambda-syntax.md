# Lambda Syntax
This document is compares the new syntax idea with Javascript syntax.

## Grammar
```bnf
lambda = "{" 
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