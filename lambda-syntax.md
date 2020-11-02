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
  = "{" {assignable ","}* expr "}"
  | "{" {"|" {assignable ","}+ expr}+ "}"
  
assignable 
  = [type] assignable'

assignable'
  = destructure
  | identifier
  
destructure
  = tagDestructure
  | recordDestructure

tagDestructure
  = tagLiteral ["(" destructure ")"]
  
application
  = expr "." "(" {[identifier "="] expr ","}* ")"
```
## Type annotation
Unlike most modern, New types are like C-like languages, where types comes before expressions. For example, instead of `x: number`, in New it's `number x`.
Reason:
1. allows better auto-complete experience in especially for record types, and destrcuturing patterns.
For example, you will get suggestions of `name` as soon as you reach the cursor.
```ts
type People = {string name, number age,}
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
3. Less boilerplate when asserting the type of the returned expression 
For example,
```ts
// In Typescript
const square = (x: number) => {
  const result: number = x * x
  return result
}

// In New
square = {
  number x,
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
  number x = 5,
  x.cos.sin
}
g = {
  number x = 6,
  x.cos.sin
}
// refactoring does not change much of the original code
h = {
  number x,
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
f = {number x, number x.plus(1)}
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
  string s,
  number from,
  number to =? s.length,
  ...
}
"Hello".slice(0)
"Hello".slice(0, 2)
```

### keywords argument
Keyword arguments does not need special syntax declartion (like OCaml), and keyword can only be specified after keywordless arguments.
```ts
// ts
f = ({a, b, c}: {a: number, b: number, c: number)) => {...}
f({a: 1, b: 2, c: 3})
// new
f = { number a, number b, ...}
1.f(2, c=3)
```

### Function application
Function application (or invocation) in New uses the universal function call syntax (UFCS) because:
1. allows IDE to suggestion autocompletion easily
2. natural function chaining

To opt-out of UFCS, we can use special expression `_` to indicate that we want to invoke a function without passing any arguments.
For example:
```ts
moreThan = { 
  number x,
  number y,
  // body
}
// The following are equivalent
2.moreThan(3)
_.moreThan(2, 3)
```

### Currying
All functions in New can be curried using `..` syntax, why this special syntax is needed? Why not just make currying the default?
Reason:
1. default currying produces cryptic error message, it's often misleading especially when user accidentally missed out some arguments
2. it's impossible to notice if a function is curried or applied completely just by reading the call-site without peeking at the function's definition

Currying examples:
```ts
f = {
  number a,
  number b,
  number c,
  a.plus(b).minus(c)
}
// a and b applied
x = {number x, 1.f(2, x)}
// same as
x = 1.f(2,..)

// a and c applied
y = {number x, 1.f(x, 3)}
// same as
y = 1.f(c=3,..)

// b and c applied
z = {number x, x.f(2, 3)}
// same as
z = _.f(b=2, b=3)
```

## Types
### Record types
```
type People = {
  name: string
  occupation: {
    name: string
    company: string
  }
}
```

### Tag types
Tag is a special kind of string (that starts with `#`) where its type is itself (in upper universe), under the hood it's just normal string. Moreover, it can be tagged with any other types.
Tag types can be used to emulate nominal types, which is useful in discriminating string values that carry different semantics.
For example, 
```ts
// Error-prone way
type People = {
  id: string,
  phoneNumber: string,
}
// Accidentally swapping id with phoneNumber won't trigger compile error
people = People {
  id= "+60123456789",
  phoneNumber= "1234",
}
```
The situtation above can be improved by using tagged types as such:
```ts
type People = {
  id: #Id(string)
  phoneNumber: #PhoneNumber(string)
}

// the following usage will result in compile error
people = People {
  id= #PhoneNumber("+60123456789"),
  phoneNumber= #Id("1234"),
}
```

### Sum Types
Sum types (or disjoint union) in New is very similar to polymorphic variants of OCaml and discriminated union of Typescript, bascially it's a union of tagged types.
Reason:
1. constructors of a union can be reused by another union without having name collision error.
2. union type can be inferred, meaning that no union types are required to be declared in advance

Example:
```ts
// TODO: the following grammar can cause ambiguity to lambda
type Color = {
  | #red
  | #green
  | #blue 
  | #yellow
}

toHex = {
  Color color,
  string color.{
  | #red, "#FF0000"
  | #green, "#00FF00"
  | #blue, "0000FF"
  | other, "unknown"
  }
}
#red.toHex.(console.log)
```
Another example:
```ts
type BinaryTree = {
  Type T,
  {
    #Leaf,
    #Node({
      element: T,
      left: T.BinaryTree,
      right: T.BinaryTree
    })
  }
}

search = {
  Type T = infer,
  T.BinaryTree tree,
  T value,
  Boolean tree.{
  | #Leaf, 
      #False
  | #Node({element, left, right}),
      element.equals(value).{
      | #True, 
          #True
      | #False,
          left.search(value).or(right.search(value))
      };
  }
}
```

## Pattern Matching
Pattern matching is done with branched function, where the pipe `|` signifies a new branch, for example:
```ts
type Boolean = {| #True | #False}
(Boolean, Boolean).to(Boolean) or = {
| #False, #False, #False
| _, _, #True
}

// Usage
#True.or(#False)
```
## Monadic binding
Sometimes, we only care about the happy path of a program and we want to ignore the bad path, but if the function is non-trivial, we can end up in a very deeply nested pattern match.
For example:
```ts
computeBounds = {
    number.array xs,
    sorted = xs.sort,
    sorted.head.{
    | #Some(lower), 
        sorted.last.{
          | #Some(upper),
              #Some({lower, upper})
          | #None
        }
    | #None
  }
}
```
We can rewrite the code above as below to improve the readability, it is very similar to Haskell do-notation, but not as powerful.
```ts
computeBounds = {
  number.array xs,
  sorted = xs.sort,
  #Some(lower) = sorted.head, // <- The inferred return type contains #None
  #Some(upper) = sorted.last,
  #Some({lower, upper})
}
```
How about in the case where we need to call different functions but each of them returns different types?  
Suppose the same example as above but `head` returns `Option` but `last` returns `Result`. 
In languages that uses non-polymorphic variants types, you will need to convert one of them to conformed to the same type, say converting `Option` to `Result`.  
Fortunately, in New, you don't have to do all these conversions because variants are polymorphic, meaning that it's ok that the return type can be variants from either `Option` or `Result`.
For example:
```ts
computeBounds = {
  number.array xs,
  sorted = xs.sort,
  #Some(lower) = sorted.head, // <- The inferred return type is now {| #None }
  #Ok(upper) = sorted.last, // <- The inferred return type is now {| #None, | #Error(String)}
  #Some({lower, upper}) // <- The inferred return type is now {| #None, | #Error(String) | #Some({lower: number, upper: number})}
}
```
Say we use the same example above, but we want to remove the `#Error` variant from the returned result, we can do as such:
```ts
computeBounds = {
  number.array xs,
  sorted = xs.sort,
  #Some(lower) = sorted.head, 
  #Some(upper) = sorted.last.{
  | #Ok(upper), #Some(upper)
  | #Error(_), #None
  }, 
  number.Result #Some({lower, upper}) 
}
```
