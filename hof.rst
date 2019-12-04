Higher-Order Functions
======================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/hof.key.pdf>`_

**Quizzes**

cse116-map-ind -> D

cse116-quiz-ind -> D

cse116-foldeval-ind -> B

cse116-foldtype-ind -> D

cse116-foldl2-ind ->

In this lecture: code reuse with higher-order functions (HOFs)

e.g.: map, filter, fold

Recursion
^^^^^^^^^
Gets pretty old pretty quickly!

Ex. a function that finds all even nums in a list

.. code-block:: haskell

    evens :: [Int] -> [Int]
    evens []                      = []
    evens (x:xs) | x `mod` 2 == 0 = x:(evens xs)
                 | otherwise      = evens xs

or a function that filters 4 letter words

.. code-block:: haskell

    fourChars :: [Int] -> [Int]
    fourChars []     = []
    fourChars (x:xs) | (length x) == 4 = x:(fourChars xs)
                     | otherwise       = fourChars xs

HOFs
^^^^

HOFs are a general pattern expressed as a HOF that takes customizable args, applied multiple times

Filter
------

.. code-block:: haskell

    filter :: (a -> Bool) -> [a] -> [a]  -- polymorphic type!
    filter f [] = []
    filter f (x:xs)
        | f x       = x:(filter f xs)
        | otherwise = filter f xs

    -- now we can:
    evens = filter isEven
        where isEven x = x `mod` 2 == 0

    fourChars = filter isFour
        where isFour x = length x == 4

.. _map:

Map
---
Ex: we want to do some op on every elem

.. code-block:: haskell

    -- boring!
    shout []     = []
    shout (x:xs) = toUpper x : shout xs

    square []     = []
    square (x:xs) = x * x : square xs

Let's do this!

.. code-block:: haskell

    map :: (a -> b) -> [a] -> [b]
    map f []     = []
    map f (x:xs) = f x : map f xs

    -- so
    shout = map (\x -> toUpper x)
    square = map (\x -> x*x)

Fold
----

Ex: length/sum of a list

How about joining a list of strings?

.. code-block:: haskell

    cat :: [String] -> String
    cat []     = ""
    cat (x:xs) = x ++ cat xs

Fold-Right
^^^^^^^^^^
This is fold-right!

.. code-block:: haskell

    foldr :: (a -> b -> b) -> b -> [a] -> b
    foldr f b []     = b
    foldr f b (x:xs) = f x (foldr f b xs)

    -- so:
    sum = foldr (+) 0
    cat = foldr (++) ""
    len = foldr (\x n -> 1 + n) 0

It's called this because it accumulates from the right (expansion is right associative)

Fold-Left
^^^^^^^^^
What about tail recursive versions?

.. code-block:: haskell

    -- tail recursive cat!
    catTR :: [String] -> String
    catTR xs = helper "" xs
        where
            helper acc []     = acc
            helper acc (x:xs) = helper (acc ++ x) xs

so:

.. code-block:: haskell

    foldl :: (a -> b -> b) -> b -> [a] -> b
    foldl f b xs = helper b xs
        where
            helper acc []     = acc
            helper acc (x:xs) = helper (f acc x) xs

    -- so, syntax is the same as foldr:
    sumTR = foldl (+) 0
    catTR = foldl (++) ""

Flip
----
Useful HOF:

.. code-block:: haskell

    -- instead of writing:
    foldl (\xs x -> x : xs) [] [1, 2, 3]

    -- write:
    foldl (flip (:)) [] [1, 2, 3]

    flip :: (a -> b -> c) -> (b -> a -> c)

Compose
-------
.. code-block:: haskell

    map (\x -> f (g x)) ys
    -- ==
    map (f . g) ys

    (.) :: (b -> c) -> (a -> b) -> a -> c
