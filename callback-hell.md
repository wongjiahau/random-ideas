# Curing Callback Hell without Sugar

## Preface
I had been trying to tackle the callback-hell (or indentation hell) problem for a long time.

Basically, how can I structure the grammar of my language such that
callback-hell will not even exist in the first place?

And today, out of a sudden, I finally discovered a *sugarless* solution. In other words, no
syntax sugar required (T&C applied)!

## Intro: What is callback hell?
Most Node.js programmers that started before [async/await
syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
will be familiar with this abomination:
```js
getApple((err, apples) => {
  if(err) return err
  getBanana((err, bananas) => {
    if(err) return err
    sendFruits(pack(apples,bananas), (err) => {
      if(err) return err
      else return true
    })
  })
})
```

Basically, it is a pyramid of callback functions which is **hard to read** due to:
1. Descending indentations
2. Lots of parentheses

These problem is not just unique to JavaScript, therefore other languages also
has their own remedies.

Though these remedies are fine, they are mostly made of syntactic sugar, which 
some language designers might find it nauseous (at least for me), because
there's only so much sugar that you can stuff before your language is causing
diabetes (decision fatigue) to the user.

We will go through each of the remedy, and finally I'll present to you the
*sugarless* solution.

## Not really a remedy: Promise API
Promise is actually not a syntactic sugar, it is just an API. 

However, it is necessary for the understanding of the following remedies.
With promise, the code above looks like (after
[promisification](https://javascript.info/promisify)):
```js
getApple.then(apples => 
  getBanana.then(bananas => 
    sendFruits(pack(apples, bananas))
  )
)
```
Though the code looks shorter where error handling is implicit, 
we can still see that this will eventually descend into callback hell, though
slower.


## Remedy 1: Async/await
The most popular solution is probably async/await, pioneered by C#, and
eventually adopted by a lot of languages including JavaScript.

With async/await, the aforementioned code can be transformed into (assuming we
promisify the functions):
```js
let apples = await getApple
let bananas = await getBanana
await sendFruits([apples, banana])
```
Though not crucial to the discussion, it is worth mentioning that async/await
is not just a statement-wise sugar, it is expression-wise.

That is to say, we can write this instead:
```js
await sendFruits([await getApple, await getBanana])
```

## Remedy 2: Do notation
[Do notation](https://en.wikibooks.org/wiki/Haskell/do_notation) is a syntactic
sugar invented by the designers of Haskell. 

For example,
```hs
main = do 
  apples <- getApple 5
  bananas <- getBanana 2
  return (sendFruits [apples, banana])
```
Though it works, it has a few problems, most notably it produces very cryptic
error messages, and it is super confusing because you have to understand Monad
which requires understanding typeclass... you probably have to learn most of
Haskell to really understand do-notation. 

## Remedy 3: Koka with-statement
With-statement is an elegant syntactic sugar, where it cleverly juggles the placement of lambdas.
For example,
```kk
pub fun test-with2() {
  with x <- list(1,10).foreach
  println(x)
}
```
is desugared into:
```kk
list(1,10).foreach(fn(x) { 
  println(x) 
}).
```

## Remedy 4: Koka effect handlers
The most powerful remedy of all is probably Effect Handlers. It is not just
syntactic sugar, but a whole new set of mechanisms for defining control flow.

Since it is a monster by itself, I will do no justice by summarizing it here,
thus I encouraged you to read more about it
[here](https://koka-lang.github.io/koka/doc/book.html#why-handlers).

## Finale: The Sugarless Solution
This solution assumes that the language syntax has these two rules:

### 1. Left associative binary function/operation
This means:
- `a b c` is equivalent to `b(a, c)`
- `a b c d e` is equivalent to `(a b c) d e`, which is equivalent to `d(b(a, c), e)`

### 2. Inverted lambda syntax
Usually, the lambda syntax has these characteristics in most languages:
- Parameter comes before body, for example `parameter -> body`
- The parameter-body separator is right-associative, for example `a -> b -> c` means `a -> (b -> c)`

Inverted means that:
- Body comes before parameter, for example `body <- parameter`
- The parameter-body separator is left-associative, for example `c <- b <- a` means `(c <- b) <- a`

By naively translating the promise example (note that the arguments of `then`
is flipped), we will get this:
```
[apples, bananas] |> sendFruits
  => apples then getApple
  => bananas then getBanana
```
With explicit parentheses:
```
(((([apples, bananas] |> sendFruits)
  => apples) then getApple)
  => bananas) then getBanana
```
By renaming `=>` to `where`, and `then` to `await`, we'll get a more sensical code:
```
[apples, bananas] |> sendFruits
  where apples await getApple
  where bananas await getBanana
```
And notice how there is no callback-hell at all, and thus no remedies are required!

Of course, this syntax is not without downsides, most notably user have to read
from bottom to top to follow the order of evaluation.
