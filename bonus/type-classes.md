## Type Classes: Ad-hoc overloading

If a type contains a type variable, the function behaves the same for
all types. In other words: we can not assume anything about the
type. In a lot of cases, we do not want to handle all types, but only
those that support certain operations.

What is the most general type of our `elem` function? More than just
integers. But can it handle *all* types?

~~~
:t elem
elem :: Eq a => a -> [a] -> Bool
~~~

`elem` can handle all types of list elements that support equality.

Types that support equality are in the *type class* `Eq`. The type of
`elem` contains a *class constraint*.

Read: For all types `a` that support equality...

Type class: a collection of types that support common functionality.

The comparison operator is actually a *method* of the type class `Eq`.

~~~
> :t (==)
-- Accepts any type that is an instance of Eq

> :info Eq
~~~

Haskell brings a number of standard type classes. Defining your own
type classes in a Haskell program is very common.

~~~
> :i Ord
> :i Show
> show 3
> show "4"
> import Data.Aeson
> :info ToJSON
> :info FromJSON
~~~


## Class and Instance Definitions

Assume you are unhappy with the fact that you can't write:

~~~
if 0 then ... else ...
~~~

because `0` is an `Int`, not a boolean.

This is arguably a bad idea in the first place, and you really want to
just have booleans in conditions.

But if you must do it, type classes can help you.

~~~
class Boolsy a where
    boolsy :: a -> Bool
~~~

~~~
if boolsy 0 then ...
if boolsy "" then ...
if boolsy False then ...
~~~

~~~
instance Boolsy Bool where
    boolsy True  = True
    boolsy False = False

instance Boolsy Int where
    boolsy 0 = False
    boolsy _ = True
~~~

~~~
if' :: Boolsy c => c -> a -> a -> a
if' c thn els = if boolsy c then thn els
~~~
