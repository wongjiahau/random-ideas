# KK language
Why the name KK?
- easy to type
- easy to pronounce
- audibly distinguishable

This document is compares the KK syntax with Typescript/Haskell's syntax.
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
  = "(" {assignable ","}* expr ")"
  | "(" {{assignable ","}+ expr ";"}+ ")"
  
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
  = expr "." "(" {[identifier ":"] expr ","}* ")"
```
## Type annotation
Types comes before variable name like Typescript but without colon. Reason:
1. it's more readable for defining nested record type
For example: 
```hs
type People = {
  name String,
  job {
    company String,
    salary Float
  }
}
```
2. it's more familiar for Typescript user
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
f = (
  _,
  x Number = 5,
  x.cos.sin
)
g = (
  _,
  x Number = 6,
  x.cos.sin
)
// refactoring does not change much of the original code
h = (
  x number,
  x.cos.sin
)
f = (_, 5.h)
g = (_, 6.h)
```
### No argument
There's no function without argument in New, if a function has no argument it's evaluted immediately, the following function in JS and New are equivalent.
```js
// js
const f = (() => {
  const x = "hello"
  const y = " world"
  return x + y
})

// new
f = (
  x = "hello",
  y = "world ",
  x.concat(y)
)
```
In order to create a function without argument, you can use the special variable `_`:
```ts
f = (
  _,
  x = "hello",
  y = "world "
  x.concat(y)
)

// To invoke it
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
slice = (
  string s,
  number from,
  number to =? s.length,
  ...
)
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
f = (number a, number b, ...)
1.f(2, c: 3)
```

### Function application
Function application (or invocation) in New uses the universal function call syntax (UFCS) because:
1. allows IDE to suggestion autocompletion easily
2. natural function chaining

To opt-out of UFCS, we can use special expression `!` to indicate that we want to invoke a function without passing any arguments.
For example:
```ts
moreThan = ( 
  number x,
  number y,
  // body
)
// The following are equivalent
2.moreThan(3)
!moreThan(2, 3)
```

### Problems
Invoking a function of a property of a record becomes weird:
```ts
db = {
  findUser: (
    string id,
    ...
  )
}
// the following is incorrect
db.findUser(id: "123")

// The correct version should be
!db.findUser(id: "123")
```
*Note: the syntax above needs to be solved, if not it's very awkward*
### Swapping argument position
By default, in the UFCS notation, the first argument binds with the topmost variable. However, we can make the first argument to bind with other variable using keyword arguments.
For example, suppose we have a minus function:
```ts
minus = (
  number left,
  number right,
  ...
)
```
Then following are equivalent:
```
9.minus(2)
9.minus(left: 2)
2.minus(right: 9)
```

### Currying
*Note to self: is this necessary?*
All functions in New can be curried using `..` syntax, why this special syntax is needed? Why not just make currying the default?
Reason:
1. default currying produces cryptic error message, it's often misleading especially when user accidentally missed out some arguments
2. it's impossible to notice if a function is curried or applied completely just by reading the call-site without peeking at the function's definition

Currying examples:
```ts
f = (
  number a,
  number b,
  number c,
  a.plus(b).minus(c)
)
// a and b applied
x = (number x, 1.f(2, x))
// same as
x = 1.f(2,..)

// a and c applied
y = (number x, 1.f(x, 3))
// same as
y = 1.f(c=3,..)

// b and c applied
z = (number x, x.f(2, 3))
// same as
z = !f(b: 2, c: 3)
```

## Types
### Record types
*Dilemma: if types come before variable, record type will look very awkward.*
*Another note: I guess it's fine, it can be weird either way*
```
type People = {
  String name,
  {
    String name,
    String company
  } occupation,
}
```
*Note: there's a problem, how do we know the above syntax means an object or a function?*
### Function types
Function type is a list of argument-type pairs, where the last pair represent the return type.
```ts
// a and b is argument, c is return type
(\number a, number b, number c) 
```
Type parameter is inside angular bracket, for example:
```ts
<T>(Array<T> array, int length)
length = (array, ...)
```

### Tag types
Tag is a special kind of string (that starts with `#`) where its type is itself (in upper universe), under the hood it's just normal string. Moreover, it can be tagged with any other types.
Tag types can be used to emulate nominal types, which is useful in discriminating string values that carry different semantics.
For example, 
```ts
// Error-prone way
type People = {
  string id,
  string phoneNumber,
}
// Accidentally swapping id with phoneNumber won't trigger compile error
people = People {
  id: "+60123456789",
  phoneNumber: "1234",
}
```
The situtation above can be improved by using tagged types as such:
```ts
type People = {
  #Id(string) id,
  #PhoneNumber(string) phoneNumber
}

// the following usage will result in compile error
people = People {
  id: #PhoneNumber("+60123456789"),
  phoneNumber: #Id("1234"),
}
```

### Sum Types
Sum types (or disjoint union) in New is very similar to polymorphic variants of OCaml and discriminated union of Typescript, bascially it's a union of tagged types.
Reason:
1. constructors of a union can be reused by another union without having name collision error.
2. union type can be inferred, meaning that no union types are required to be declared in advance

Example:
```ts
type Color = {
  #red,
  #green,
  #blue,
  #yellow,
}

{Color color, return string}
toHex = {
  Color color,
  string color.{
    #red, "#FF0000";
    #green, "#00FF00";
    #blue, "0000FF";
    other, "unknown";
  }
}
#red.toHex.(console.log)
```
Another example:
```ts
type BinaryTree = <T>{
| #Leaf
| #Node({
    T element,
    T.BinaryTree left,
    T.BinaryTree right
  })
}

{
  Type T,
  T.BinaryTree tree,
  T value,
  return Boolean
}
search = {
  Type T,
  T.BinaryTree tree,
  T value,
  Boolean tree.{
  |#Leaf, 
    #False
  |#Node({element, left, right}),
    #False = element.equals(value),
    left.search(value).or(right.search(value))
  }
}
```

## Pattern Matching
Pattern matching is done with branched function, where the pipe `|` signifies a new branch, for example:
```ts
type Boolean = {| #True | #False}
{Boolean, Boolean, Boolean} 
or = {
  | #False, #False, #False
  | _, _, #True
}

// Usage
#True.or(#False)
```
## Monadic bindings
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
We can rewrite the code above as below to improve the readability using monadic bindings:
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
{number.array, {number lower, number upper}.Option}
computeBounds = {
  number.array xs,
  sorted = xs.sort,
  #Some(lower) = sorted.head, 
  #Some(upper) = sorted.last.{
  | #Ok(upper), #Some(upper)
  | #Error(_), #None
  }, 
  #Some({lower, upper}) 
}
```
### Should we use monadic bindings?
Yes, in fact it's encouraged, because it makes the resulting code less nested thus much more readable, which is the idea behind [Railway Programming](https://fsharpforfunandprofit.com/rop/) (some personal opinion: Railway Programming is actually not easy in most functional languages because their variants are closed by default, in other words you cannot combine variants from different union to form a new union implicitly).  

This programming method is also especially useful for prototyping, because to make a proof-of-concept (POC), usually only the happy path is considered.  
In language like Haskell or Rust where you are forced to handle the sad path, either you really handle them (which is worthless for POC) or you throw a bomb at those branch, i.e., terminate the program when those branch are hit.  
However, with the flexibilty of variants in New, you don't have to plant bomb everywhere, what you need to do is just to let them bubble up, and handle them explicitly.
The main advantage of this approach is that you won't have to worry your production server terminating when those unhandled branches are hit. Nevertheless, it means that its harder to trace exactly where the unhandled branches are.

### Monadic binding (edge cases)
In the case where the return types of the non-happy contains types with the same variant but different payload type, it will be a compile error. 
For example, the following code is invalid:
```ts
getFortune = {
  x = .random,
  x.mod(2).equals(1).{
    | #True, #Surpise(x)
    | _, #Nothing
  }
}
getSurprise = {
  x = .random,
  x.mod(2).equals(1).{
    | #True, #Surpise("You got a present")
    | _, #Nope
  }
}
main = {
  #Nothing = .getFortune,
  #Nope = .getSurprise,
  #NoLuck
  // Error, #Surprise(number) and #Surprise(string) cannot form a tagged union,
  // because number and string cannot be discriminated
}
```
To resolve this problem, we can simply rewrap one of the `#Surprise` result to another tag.
```ts
main = {
  #Nothing = .getFortune.{
    #Surprise(x), #FortuneSurpise(x)
  },
  #Nope = .getSurprise,
  #NoLuck
}
```
To resolve this problem, we can simply rewrap one of the `#Surprise` result to another tag.
```ts
main = {
  #Nothing = .getFortune.{
    #Surprise(x), #FortuneSurpise(x)
  },
  #Nope = .getSurprise,
  #NoLuck
}
```

Transpiled Javascript for the code snipper above (for compiler devs):
```js
function getFortune() {
  const x = random();
  const result = equals(mod(x, 2), 1);
  switch(result.$) {
    case 'True':  return {$: 'surprise', _: x}
    default:  return {$: 'Nothing'}
  }
}
function getSurprise() {
  const x = random()
  const result = equals(mod(x, 2), 1)
  switch(result.$) {
    case 'True':  return {$: 'Surprise', _: "You got a present"}
    default:  return {$: 'Nope'}
  }
}
function main {
  const v1 = getFortune()
  switch(v1.$) {
    case 'Nothing': {
      const v2 = (() => {
        const v2 = getSurprise()
        switch(v2.$) {
          case 'Surprise': return {$: 'FortuneSurprise', _: $._}
          default: return v2
        }
      })()
      switch(v2.$) {
        case 'Nope': return {$: 'NoLuck'}
        default: return v2
      }
    }
    default:
      return v1
  }
}
```

### Unhandled cases
In New, it's not necessary to handle all cases, but that doesn't mean the type system is unsound, because all unhandled case will just return the same value. This is possible because the variants are polymorphic.  
For example:
```ts
type Shape = {
  | #Dot
  | #Circle({radius})
}

area = {
  Shape shape,
  shape.{
    #Circle({radius}), #Some(radius)
  }
}
```
The above `area` function does not handle the `#Rectangle` variant, but there's no type error, simply because the inferred return type of the function is `{|#Some(number) | #Dot}`.  

However, if the `area` function is re-written as below, then there will be a compile error, because only tagged union is allow.
```ts
area = {
  Shape shape,
  shape.{
    #Circle({radius}), radius
    // ERROR: cannot form a tagged union with "number"
    // TIP: consider wrapping "number" with a tag,
    //      For example, "#Some(number)"
  }
}
```
Why is this **unhandled cases** feature useful?  It's especially useful when we want to handle the non-happy path explicitly in monadic bindings.

Say the function below that does not utilise this feature:
```ts
getNetPrice = {
  Item item,
  #Ok(sellingPrice) = item.getSellingPrice.{
    #Error(_), #None,
    other, other
  },
  #Ok(grossPrice) = item.getGrossPrice.{
    #Error(_), #None,
    other, other
  }
}
```
By utilising this feature, the `other, other` branch is unnecessary:
```ts
getNetPrice = {
  Item item,
  #Ok(sellingPrice) = item.getSellingPrice.{
    #Error(_), #None,
  },
  #Ok(grossPrice) = item.getGrossPrice.{
    #Error(_), #None,
  }
}
```


## Promise
Code needs to be transpiled into continutation passing style (CPS).
Transpiled JS:
```ts
// Typescript
type Result<T, E> = {
  $: 'resolved',
  value: T
} | {
  $: 'rejected',
  error: E
}
const runPromise = <T, E>(
  promise: Promise<T,E>,
  f: Result<T, E>
) => {
  promise
    .then(value => f({$:'resolved', value}))
    .catch(error => f({$: 'rejected', error})
}
```
## References
1. Pattern Calculus. https://www.cas.mcmaster.ca/~kahl/Publications/Conf/Kahl-2004a.pdf