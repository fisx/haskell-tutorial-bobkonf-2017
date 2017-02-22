# Introductory Haskell, bobkonf'17

These are the course materials for http://bobkonf.de/2017/fischmann.html.

Based on earlier tutorials with Andres Löh, Alexander Ulrich.


## Getting things running

- open `Main.hs` from this repository in an editor (e.g., vim).
- open a terminal window and enter `ghci Main.hs`.
- try this:

```
*Main> hellobobkonf
seems like you're all set!
```


## Basic data types

~~~
> 42
> 2.3
> "foo"
> 'x'
> True
> False
> (42, "foo")
~~~


## Basic Syntax

~~~haskell
-- A function named 'f' with one parameter
greet you = "hey, " ++ you
double x = x * 2

-- A function named 'g' with two parameters
mul x y = x * y
greetWith greeting you = greeting ++ ", " ++ you
~~~

~~~
-- Function application is syntactically lightweight.
> mul 42 (double 23)
~~~

More forms of expressions:

~~~
-- let-bindings
> let x = 5 in x + 42

-- conditionals
> if x == 5 then [42] else []

-- lambda expression
> \x y -> x + y
~~~

The last one is an *anonymous function* or *lambda
expression*. Functions are first-class values - regular values that you
can pass around like any other.


## Inductive list definition, list construction examples

Lists are an important data structure in Haskell (as in most other
functional languages).

Lists are defined *inductively*. You might know the schema from
various Lisp or ML dialects.

~~~
-- The empty list
> []

-- Construct a new list from a value (head) and a list (tail): cons
> 2 : []

-- Constructing lists with more elements
> 1 : (2 : [])

-- Without parentheses
> 1 : 2 : []

-- Syntactic sugar for list construction
> [1, 2, 3]
> 42 : [1, 2, 3]
> [42, 19] ++ [1, 2, 3]
~~~

Syntax `[e_1, ..., e_n]` is transformed into an equivalent expression
using the cons `(:)` operator.


## Functions on lists


### list length

*Deconstruct* the list according to its inductive definition by
 *pattern matching*. A list is either empty or a cons combination of a
 value and some other list. Treat the cases separately!

~~~haskell
length []        = 0
length (hd : tl) = 1 + length tl
~~~

~~~
length [1..3]           ==
length (1 : 2 : 3 : []) ==
1 + length (2 : 3 : []) ==
1 + 1 + length (3 : []) ==
1 + 1 + 1 + length ([]) ==
1 + 1 + 1 + 0           ==
3
~~~

* Two equations, two cases for the underlying data type: case-wise
  function definition. Very common pattern for function definitions
* A *recursive* call deals with the tail of the list (if necessary)
* **boolean or: short-cut evaluation**
* In Haskell, all bindings are mutually recursive by default


### list membership

~~~haskell
-- Any value is certainly not an element of the empty list
elem x [] = False

-- Check wether the head value equals 'x' or if 'x' occurs
-- in the tail of the list.
elem x (y : ys) = x == y || elem x ys
~~~

~~~
> elem 5 [6, 9, 42]
> elem 9 [6, 9, 42]
~~~


## Truth values

Truth values are just a data type with two constructors:

~~~
> True
> False
~~~

Define boolean functions by pattern-matching on those constructors. We
define our own version of boolean that behaves exactly like the
Haskell one. The `or`-operator is present in the standard library, but
we define our own version.

Haskell allows to define our own infix operators.

~~~haskell
True  || y = True
False || y = y
~~~

~~~
> True  || False
> False || False
~~~

Side note: An infix operator can be used as a regular function:

~~~
> (||) True False
~~~

Additionally, any function with two arguments can be used as an infix
operator by enclosing it in backticks.

~~~
> 5 `elem` [1, 2, 3, 5]
~~~


## Lazy Evaluation

Short-cut behaviour of boolean operators is not a special hack in Haskell.

Lazy evaluation: function arguments are evaluated only when they are
actually required. Usually, *required* means that they are matched on.

~~~
-- A special value that raises an exception when it is evaluated
> undefined

-- Only need to evaluate the first argument to True to give the result
> True || undefined

-- Need to evaluate the second argument.
> False || undefined
~~~

Lazy evaluation is neat:

~~~
> let allIntegers = [1..]
-- Show evaluation status of binding. Thunks are marked with an underscore
> :print allIntegers
> :t take
> take 10 allIntegers
> :print allIntegers
> :t sum
> sum (take 10 allIntegers)
> sum allIntegers
~~~

We can work with non-terminating computations:

~~~haskell
ones = 1 : twos

twos = 2 : ones
~~~


## Type Inference

Haskell is a strong and statically typed language. The compiler checks
if every expression is type-correct.

*But*: We have not seen any type signatures so far. How is this
 consistent with static typing?

*Answer*: The compiler can infer the type of an expression from the
 types of its sub-expressions. *Type inference*

Example:

~~~haskell
not True  = False
not False = True
~~~

From the code, we can conclude that:

* It's a function (it has an argument)
* It takes a truth value
* It produces a truth value

~~~haskell
not :: Bool -> Bool
~~~

The compiler checks wether we have given a correct type for the
function.

We can also ask the compiler for the type of some expression or
function:

~~~
> :t not
> :t not True
~~~

What is the type of the "or" function?

~~~
> True || False
~~~

It takes two bools and produces a bool:

~~~haskell
(||) :: Bool -> Bool -> Bool

-- Actually: A function that is applied to a bool and gives us another function.
(||) :: Bool -> (Bool -> Bool)
~~~

Let's return to our `elem` function. We can give its type as follows:

~~~haskell
elem :: Int -> [Int] -> Bool

-- But actually, it's this type:
elem :: Int -> ([Int] -> Bool)
~~~

What is the type of `elem 5`?

Functions with more than one parameter can be *partially
applied*. Partial application specializes (or fixes) a function on
some parameters.

~~~haskell
containsFive :: [Int] -> Bool
containsFive = elem 5
~~~

~~~
> containsFive [1, 2, 3]
> containsFive [1..10]
~~~


## (Parametric) Polymorphism

Let's define a function that appends two lists of integers.

~~~haskell
(++) :: [Int] -> [Int] -> [Int]

-- Case 1: first argument is the empty list
[] ++ ys     = ys

-- Case 2: non-empty first argument.
(x:xs) ++ ys = x : (xs ++ ys)
~~~

Do we use the fact that we append lists of *integers*
specifically? Wouldn't the code for appending lists of strings look
exactly the same (except in the type signature)?

==> Remove type signature

~~~
["foo", "bar"] ++ ["baz"]
~~~

What, then, is the type of `(++)`? Let's ask the compiler. For any
program, Haskell infers not only *some* type, but *the most general
type*.

~~~
:t (++)
(++) :: [a] -> [a] -> [a]
~~~

Read this as: For *any element type* `a`, the function takes two lists
of `a` and produces a new list of `a`. Crucially though, both
arguments must have *the same* element type.

`a` is a type variable which can be *instantiated* to any type.

*Parametric polymorphism*: A function behaves the same, regardless of
the type. Parametric polymorphism is a powerful way to write abstract
and generic code.

Another polymorphic function

~~~haskell
map :: (a -> b) -> [a] -> [b]
map f (x:xs) = f x : map f xs
map _ []     = []
~~~

Note also that this is a higher-order function.  It takes another
function as an argument.

~~~
> map (elem 5) [[1..10], [2..4], []]
> map (+ 1) [1, 2, 3, 4, 5]
~~~


## Data types

Haskell supports a powerful mechanism to define new data types:
*algebraic data types* or *sums of products*.

Simple example:

~~~haskell
data Maybe a = Just a | Nothing
~~~

~~~
> Just 3
> Nothing
> [Just 3, Just False]
> [Just 3, Nothing, Just 5]
~~~

~~~haskell
data Either a b = Left a | Right b
~~~

~~~
> [Left 3, Right False]
~~~

(Both `Maybe` and `Either` are `Monad`s.  Enough said.  :-)

Let's define a data type for binary trees with node labels of *some
type*. The type gets a *type parameter*.

~~~haskell
data Tree a = Leaf
            | Node (Tree a) a (Tree a)
~~~

We have two constructors. A `Leaf` is a tree, as well as a `Node`,
applied to three parameters. Note that the type is recursive: A tree
contains two child trees (if it is a `Node`).

This is an easy example of a generic data structure.

~~~
-- The 'Leaf' constructor is a valid tree for /any/ element type.
> :t Leaf
> :t Node Leaf "foo" Leaf
~~~

Side note: What is the type of `Node`?

~~~
> :t Node
~~~

Data constructors are just functions and can be treated like any other
function.

Functions over data types usually follow the structure of inductively
defined types by pattern matching and recursion (just as in the case
of lists).

~~~haskell
sumTree :: Tree Int -> Int
sumTree Leaf           = 0
sumTree (Node t1 x t2) = sumTree t1 + x + sumTree t2

t :: Tree Int
t = Node
       (Node Leaf 3 Leaf)
       91
       (Node
           (Node Leaf 16 (Node Leaf 21 Leaf))
           24
           Leaf)

sumTree t
~~~

We can interpret binary trees as *search trees*. Write a function that
checks wether a value is a member of a given search tree.

~~~haskell
elemTree :: Ord a => a -> Tree a -> Bool
elemTree _ Leaf = False
elemTree a (Node t1 x t2) =
  if a == x then True          else
  if a < x  then elemTree a t1 else
                 elemTree a t2
~~~


## Purity and Effects

*purity* or *referential transparency*:

> You can replace any two expressions by each other that evaluate to
  the same value.

> The value of a piece of code is everything that matters.

> *No effects* (widget manipulation, disk access, random data
   generation, ...)

No, wait:

> *Effects are explicit* in the types!

~~~
generateRandomNumber :: Int -> IO Int
readString           :: IO String
~~~

`IO a` is an action that may have some effects and yields an `a`.

`generateRandomNumber` always retursn the same *action* on the same
input `Int`, so referential transparency still holds!

Main entry point of every program:

```haskell
main :: IO ()
main = putStrLn "hello, world"
```


## Holes

In Haskell, the expressive type system allows to specify interesting
properties of programs. But furthermore, we can also let us be *guided
by types* during development.

~~~haskell
mapMaybe :: (a -> Maybe b) -> [a] -> [b]
mapMaybe = _
~~~

An expression that begins with an underscore is called a *hole*. The
Haskell compiler gives us information about the type of the expression
that we should fill the hole with.

Things to learn here:

- break up your problem (pattern matching), and put the pieces back
  together (make the types fit).
- `Maybe` and `[]` are related (how?).
- even with types you need to keep your brain running while coding.  :)


# Tools, Infrastructure, Further Reading


## Library search and distribution

- http://hayoo.fh-wedel.de/, https://www.haskell.org/hoogle/
    - search for stuff in any library in different ways
    - search for stuff in any library in different ways

- https://hackage.haskell.org/
    - package repository

- http://www.stackage.org/
    - started off as a version pinning tool.
    - now the stack tool stack.


## Package management

- cabal:
    - haskell package manager
    - powerful version dependency constraint solver
    - hackage-security
- stack:
    - alternative to cabal, younger and more pragmatic.
- both based on the Cabal library.


## package.cabal

~~~
name:                gorbla
version:             1.3.11
synopsis:            gorbla isn't a real project, it's just a cabal file example.
license:             AGPL-3
license-file:        LICENSE
author:              ...
maintainer:          ...
build-type:          Simple
cabal-version:       >= 1.10

library
  exposed-modules:
      Api
    , DB.Core
    ...
  build-depends:
      base >=4.7 && <4.8
    , aeson >=0.7 && <0.8
    , transformers ...
    ...

executable run-server
  main-is:
      Main.hs
  build-depends:
      base
    , acid-state
    , ...
~~~


## stack.yaml

~~~
resolver: lts-7.1

packages:
- '.'
- location: '../prelude'
- location:
    git: https://github.com/bgamari/html-parse.git
    commit: ebdd5fa3fa5a0d3c22602bec15059e821ad10ec4

extra-deps:
- react-flux-1.2.3
- ...

compiler: ghcjs-0.2.1.9007001_ghc-8.0.1
compiler-check: match-exact

setup-info:
  ghcjs:
    source:
      ghcjs-0.2.1.9007001_ghc-8.0.1:
        url: http://tolysz.org/ghcjs/ghc-8.0-2016-09-26-lts-7.1-9007001-mem.tar.gz
        sha1: e640724883238593e2d2f7f03991cb413ec0347b
~~~


## IDEs

https://wiki.haskell.org/IDEs


## Books

- Good book with online material: http://www.cs.nott.ac.uk/~pszgmh/pih.html
- Good online book: http://learnyouahaskell.com/


## Learn more

- http://stackoverflow.com/questions/tagged/haskell
- http://www.reddit.com/r/haskell
- https://ocharles.org.uk/blog
- http://haskellcast.com/
- irc: freenode \#haskell-beginners, \#haskell
- https://github.com/NicoleRauch/FunctionalProgrammingForBeginners


## Consulting / Training

- http://well-typed.com/blog
