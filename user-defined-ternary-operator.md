## Introduction
Ternary operator is a very useful syntactical notation
for condensing meaning. Basically it takes three
arguments, and syntactically speaking it's identifier is
split into two parts.

For example:  
1. Javascript conditionals `a ? b : c` 
2. Math range check `a < b < c`  
3. SQL joins `a JOIN b ON c`

This is similar to why we
prefer **infix binary operators** (IBO), because they can be
chained together easily (or so called
[*associativity*](https://en.wikipedia.org/wiki/Operator_associativity)).

For example, most if not all of us will think that A is easier to read and understand than B:

|A|B|  
|--|--|  
|`a + b + c` | `(+ (+ a b) c)` | 

Because IBO are so important,
most functional programming (FP) languages like Haskell
allow users to define custom IBO.  

For example:
```hs
-- pipe forward operator
(|>) :: a -> (a -> b) -> b
a |> f = f a

-- example usage
main = [1, 2, 3] 
  |> map (+2) 
  |> filter (>3) 
  |> print 
  |> putStrLn -- [4, 5]
```
See more at [Haskell Infix Operator](https://wiki.haskell.org/Infix_operator).


## Emulating with binary operator
However, when it comes to user-defined ternary operator,
none of the programming languages provide the mechanism.  

Although ternary operator can be emulated using double
IBO, it is not without its own caveats. 

Take [this example](https://wiki.haskell.org/Ternary_operator) from Haskell:

```hs
data Cond a = a :? a

infixl 0 ?
infixl 1 :?

(?) :: Bool -> Cond a -> a
True  ? (x :? _) = x
False ? (_ :? y) = y

test = 1 < 2 ? "Yes" :? "No"
```
The caveat of such emulation is that partial usage is
not only syntactically valid, but also
semantically valid.  

For example, we can omit that `:?`
part, and the compiler will **not** complain.
```hs
x = 1 < 2 ? "Yes"
```
From the user experience (UX) perspective, this is very bad,
because we intended users to always use `?`  together with
`?:`.

## Emulating using mixfix operators
Other than using IBO, we can also emulate ternary operators using mixfix operators.

For example, in [Agda](https://agda.readthedocs.io/en/v2.6.2/language/mixfix-operators.html): 

```hs
-- Example function name _if_else_ 
-- (emulating Python's conditional operator)
_if_else_ : {A : Set} -> Bool -> A -> A -> A
x if true else y = x
x if false else y = y
```
The caveat of such approach is that users will be
required to lookup the syntax before they can even parse
a piece of code that is filled with mixfix operators.

Because, for example, the above `_if_else_` operators can also be defined as:
```hs
if_then_else_ : {A : Set} -> Bool -> A -> A -> A
if x then true else y = x
if x then false else y = y
```
In this case, the user will not be able to parse the following code correctly without knowing the syntax of `if` or/and `else`:
```hs
x = a if b else c -- is this correct?
y = if a then b else c -- or this?
```

## The Question
Due to the aforementioned caveats of emulating ternary
operators, I am on a quest for searching a **mechanism** to allow
user-definer ternary operators that is:  
1. Unambiguous (parsable by both machine and human)
2. Universal (can be applied to all kinds of ternary operators)

## Inspiration
I have been contemplating this question for a while, but
still there is no significant progress, until I read the
book [The Relational Model of Database
Management](https://www.amazon.com/Relational-Model-Database-Management-Version/dp/0201141922)
by the inventor of Relational Algebra, [Edgar. F. Codd](https://en.wikipedia.org/wiki/Edgar_F._Codd).


His notation for denoting theta-joins is ingenious. 

For example,

|SQL|Edgar's notation|  
|--|--|  
|`X join Y on A join Z on B `|`X [A] Y [B] Z`|

The ingenious part about this is that he treats ternary
operator as **decorated binary operators**! 

In the above example, the binary operator `[]`, is
decorated/tainted with `A`, giving it a different
meaning.

This actually also relates to the mathematical notation of theta-join:

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hzoqn69hync9z2ryoff4.png)

In this case, thetha symbol is decorating the join
symbol, turning it into a ternary operator that still
behave as a binary operator.

The advantage of this approach is that ternary
operators behave like binary operators, which means
that they can be chained together naturally.

## The Potential Answer
By expanding on the idea where ternary operator are just
decorated/tainted binary operators, I first have this idea:

Ternary operators can be defined using the following syntax:

```
a X[ b ]Y c
```
Where `a`, `b` and `c` are the arguments, and `X[` and
`]Y` are together the name of the ternary operators.

For example, let's define range-check:

```hs
-- Definition
a <[ b ]< c = a < b && b < c

-- Usage
print (1 <[2]< 3) -- true
print (1 <[3]< 2) -- false
```

Assuming `[` `]` is not used anywhere in the syntax of
the language, such ternary operators usage can be parsed
easily. Because whenever the user or machine sees `[` or
`]`, then they can anticipate a ternary operator.


However, the above syntax is obviously too noisy, so to
reduce the noise, we can swap the square brackets `[`
`]`, with one of the most invisible ASCII operator, the backtick.

With this modification, the above range-check can be rewritten as follows:
```hs
-- Definition
a <` b `< c = a < b && b < c

-- Usage
print (1 <` 2 `< 3) -- true
print (1 <` 3 `< 2) -- true
```

Thus, the best mechanism that I can think of so far to define ternary operator is as follows:

```hs
a X` b `Y c

-- Where a, b and c are the arguments
-- while X` and `Y together is the name of the ternary operator
```
Also regarding precedence, ternary operators should have
lower precedence than binary operators, for example:

```
(a <` b + c `< d) MEANS (a <` (b + c) `< d)
```

Regarding, associativity, I will prefer
right-associativity over left-associativity, since it is
more common in most cases.

Therefore:
```
a X` b `Y c X` d `Y e = a X` b `Y (c X` d `Y e) 
```

For example, the conditional ternary operator:
```
true then` b `else c = b
false then` b `else c = c
```
Where:
```
a then` b `else c then` d `else e
```
Should conventionally mean (right-associative): 
```
a then` b `else (c then` d `else e)
```
Rather than (left-associative):
```
(a then` b `else c) then` d `else e
```



## Conclusion
Hopefully this article has provided you some inspiration
on the mechanism of user-defined ternary operators.

Thanks for reading.

 
