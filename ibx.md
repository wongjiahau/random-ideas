# IBX (Infix Binary Expression) [DRAFT]

## Preface
When I was developing my own language, I realised
its syntax keeps growing exponentially. I thought
that this is a bad idea as it can only get more
complicated, for a couple of reasons:

1. Harder to parse
2. Harder to write a formatter
3. Harder to write a language server (e.g. contex aware auto-completion) 

I then decided to look into a few simple data
configuration format like
[S-expression](https://en.wikipedia.org/wiki/S-expression),
[JSON](https://en.wikipedia.org/wiki/JSON) and
[XML](https://en.wikipedia.org/wiki/XML).

### S-Expression

S-expression is nice, however due to its [polish
notation](https://en.wikipedia.org/wiki/Polish_notation)
where every construction (e.g. function call,
variable declaration etc.) are **prefix**, I find
it very difficult to read. 

This is especially true as my primary language is
Chinese and English, where both are
[Subject-Verb-Object](https://en.wikipedia.org/wiki/Subject%E2%80%93verb%E2%80%93object) (SVO)
language. For example, in English, we say *I love you* instead of *Love I you*.  

So unless your primary language is
[Verb-Subject-Object](https://en.wikipedia.org/wiki/Verb%E2%80%93subject%E2%80%93object)
(VSO), e.g. Classical Arabic or Irish;
or
[Verb-Object-Subject](https://en.wikipedia.org/wiki/Verb%E2%80%93object%E2%80%93subject) (VOS), e.g. Malagasy, 
you should find S-expression not very easy to comprehend without some training.

For example, the factorial function in Scheme is defined as follows:
```scheme
(define factorial
  (lambda (n)
    (if (= n 0) 1
        (* n (factorial (- n 1)))))
```
Compare this to Javascript (where most operations are infix, thus more aligned to SVO):
```js
let factorial = (n) => {
  if (n === 0) 
    return 1
  else 
    return n * factorial(n - 1)
}
```

### JSON
JSON on the other hand is more flexible than
S-expression. However it has the same problem: 
emulating prefix-notation is easier than emulating
infix-notation in JSON.

For example, consider the following example of [MongoDB
Query
Language](https://docs.mongodb.com/manual/crud/#read-operations):

Note: I purposely leave out the infix syntax (such as
`{amount: {$gt: 100}}`), as most [Aggregation Pipeline
Operators](https://docs.mongodb.com/manual/meta/aggregation-quick-reference/)
are prefix.

```js
db.sales.aggregate([
  { 
    $match: {
      $expr: {
        $and: [ 
          { $gt: ["$amount", 100] },
          { $eq: ["$product_id", 1]}
        ]
      }
    }
  },
  {
    $project: { _id: 1 }
  }
])
```
Compare this to the equivalent SQL (which again, most operation are infix):

```sql
SELECT _id FROM sales
WHERE amount > 100 AND product_id = 1
```

In this case, the infix operators are `>`, `AND` and
`=`, which makes it more natural to read for reader whose
primary language is SVO/OVS-based.

Thus, it seems like JSON is like LISP, but worse.

## XML
Let's try to encode the factorial function in an
imaginary XML:
```xml
<define name="factorial">
  <lambda argument="n">
    <condition>
      <equals left="n" right=0/>
      <if-true> 1 </if-true>
      <if-false>
        <multiply>
          <variable name="n"/>
          <function-call name="factorial">
            <minus>
              <variable name="n"/> 
              1
            </minus>
          </function-call>
        </multiply>
      </if-false>
    </condition>
  </lambda>
</define>
```
As we can see, XML is just way too verbose to be a
suitable data format for emulating/encoding language
syntax.

### Conclusion
Since none of the existing data format above allows
**natural** language syntax construction, I decided to
invent my own, namely Infix Binary Expression (IBX),
pronounce as *ibex* if you may.

## IBX
The main difference of IBX from other data formats is
that it allows *natural language emulation*. Again,
*natural* as from the perspective of SVO/OVS language.

Therefore the main features of IBX are:  
1. First-class infix notation
2. Left associative

## Grammar
```ebnf
ibx 
  = terminal 
  | binary 
  | ternary

(* left associative *)
binary
  = ibx space terminal space terminal

(* left associative *)
ternary
  = ibx 
    space ternary_left 
    space ibx 
    space ternary_right 
    space terminal

ternary_left
  = identifier ternary_quote

ternary_right
  = ternary_quote identifier

ternary_quote
  = "'" (* can be other character as well depends of flavour *)

terminal
  = "(" ibx ")"
  | identifier
  | string
  | number

identifier
  (* note: one_of should be treated as an EBNF-level function *)
  = one_of("`~!@#$%^&*-=_+[]{}\|;:,./<>?qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM")
    one_of("1234567890`~!@#$%^&*-=_+[]{}\|;:,./<>?qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM")
    identifier
  | identifier ternary_quote ternary_quote identifier

string = (* follows JSON specification of string *)
number = (* follows JSON specification of number *)
space = (* follows JSON specification of ws *)
```
Reference: [JSON specification](https://www.json.org/json-en.html)


## Transformation Examples
|Rule|IBX| S-Expression | Note | 
| --|--| -- |  -- |
| Binary operator | `x f y` | `(f x y)` | -- |
| Ternary operator | `x f' y 'g z` | `(f''g x y z)` | `f''g` is a single identifier | 
| Left associative | `x f y g z` | `(g (f x y) z)`| | 
| Ternary center higher precendence | `x f' a b c 'g z` | `(f''g x (b a c) z)` | Treat `f'` as opening bracket `(`, and `'g` as closing bracket `)` |
| Ternary operator behaves like binary operator | `x f' y 'g z j k` | `(j (f''g x y z) k)` | Treat `f' ... 'g` as a binary operator |  

## Emulation Examples
|Language| Partial Snippet | IBX Emulation|
|--|--|--|
|[Neo4j Cypher](https://neo4j.com/developer/cypher/)|`(:Order)<-[:SOLD]-(e:Employee)` | `:Order <-' :SOLD '- (e : Employee)` |
|SQL Inner Join| `a JOIN b on c` | `a join' b 'on c` |
| Math Range check | `a < b < c` | `a <' b '< c` | 
| JSON Array | `[1, 2, 3]` | `1 , 2 , 3` | 
| Function call | `f(x)` | `x \|> f` or `x <\| f` |
| Method call | `x.f(y)` | `x f y` |

## Example
```
factorial = (n -> (
  1 if' n === 0 'else 
    (n - 1 |> factorial * n)
))
```
Notice that lambda is define in reversed, so `a <-
b` means `b -> a`, this is because IBX is left
associative.

## Limitations
The main limitation of IBX is that all constructions are
left-associative, this poses a problem when we try to emulate naturally right-associative constructs.

For example, in Javascript, lambda and assignment are right-associative:
```js
f = x => y => z

g = (x => (y => z))

// f and g are equivalent
```

Since IBX is left-associative, there's two way to emulate:
1. Use parenthesis
```
f = (x => (y => z))
```
2. Invert the syntax
```
z <= y <= x = f
```

In my opinion, both workarounds are not optimal as they
read worse than their original counterpart.

