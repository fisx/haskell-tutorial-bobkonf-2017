~~~
elemTree :: Ord a => a -> Tree a -> Bool
elemTree _ Leaf = False
elemTree a (Node t1 x t2) | a == x    = True
                          | a < x     = elemTree a t1
                          | otherwise = elemTree a t2
~~~

Equations of a function can be further qualified with *guards* that
test a boolean condition. A combination of `if`-conditionals would
work as well, but would be considerably less elegant.

We can build an associative map based on binary search trees.

~~~
-- A type alias
type Map a b = Tree (a, b)
~~~

A lookup function on such a map is easily defined:

~~~
-- incorrect. Need Ord constraint on a
-- lookupMap :: a -> Map a b -> b
lookupMap :: Ord a => a -> Map a b -> b
lookupMap x Leaf = ???
~~~

We have to account for the case that the key we search is not present
in the map. Other languages would likely "model" this case by
returning a NULL reference. NULL values lead to a large number of
well-known (and very expensive) problems (Tony Hoare: "my
billion-dollar mistake"). Luckily, the following isn't valid Haskell
code.

~~~
lookupMap x Leaf = NULL
~~~

`lookupMap` returns either a value of type `b` or nothing at
all. Haskell models this explicitly in a simple data type.

~~~
> :info Maybe
~~~

Maybe is defined in the Haskell standard library.

When calling a function that *maybe* returns a value, the type system
forces us to explicitly handle the error case (`Nothing`) by pattern
matching on the `Maybe` value.

We can now define the `lookupMap` function:

~~~
lookupMap :: Ord a => a -> Tree (a, b) -> Maybe b
lookupMap _ Leaf = Nothing
lookupMap x (Node t1 (a, b) t2)
    | a == x    = Just b
    | x < a     = lookupMap x t1
    | otherwise = lookupMap x t2
~~~

Side note: Note the nested pattern in the second equation.

~~~
t2 :: Map Int String
t2 = Node
       (Node Leaf (3, "drei") Leaf)
       (91, "einundneunzig")
       (Node
           (Node Leaf (16, "sechzehn") (Node Leaf (21, "einundzwanzig") Leaf))
           (24, "vierundzwanzig")
           Leaf)

> lookupMap 5 t2
> lookupMap 6 t2
~~~
