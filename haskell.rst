Haskell
=======

`Slides <https://owenarden.github.io/cse116-fall19/slides/haskell.key.pdf>`_

Quizzes
-------
cse116-pair-ind -> D

cse116-tpair-ind -> D

cse116-pattern-ind -> D

What is Haskell?
----------------

Haskell is a typed, lazy, purely functional language, with:

- types
- builtins (bools, numbers, chars)
- tuples
- lists
- recursion

Haskell v. lambda-calc:

- A program is an expression, not a sequence of statements
- it evaluates to a value, does not perform actions
- functions are first-class values
    - can be passed as args
    - can be returned from a func
    - can be partially applied
- but there are things that aren't funcs
    - variable assignments/literals
    - top level bindings

- you can also define funcs using equations

.. code-block:: haskell

    pair x y b = if b then x else y -- \x y b -> ITE b x y

- and patterns:

.. code-block:: haskell

    pair x y True  = x
    pair x y False = y

- a pattern is a variable (matches any value), or a value (matches that value)
- the above pattern is equivalent to:

.. code-block:: haskell

    pair x y True  = x
    pair x y b     = y

    pair x y True  = x
    pair x y _     = y -- wildcard: don't create binding

Guards
------
- an expression can have multiple guards (bool exp)

.. code-block:: haskell

    cmpSquare x y | x > y*y  = "bigger"
                  | x == y*y = "equal"
                  | x < y*y  = "smaller"

    -- equals to
    cmpSquare x y | x > y*y   = "bigger"
                  | x == y*y  = "equal"
                  | otherwise = "smaller"

Recursion
---------
Is built in!

.. code-block:: haskell

    sum n = if n == 0
                then 0
                else n + sum (n - 1)

    -- or
    sum 0 = 0
    sum n = n + sum (n - 1)

Variable Scope
--------------
- Top level vars have global scope

.. code-block:: haskell

    -- vars defined out of order
    message = if foo
                then "bar"
                else "baz"
    foo = True

    -- mutual recursion
    f 0 = True
    f n = g (n - 1)

    g 0 = False
    g n = f (n - 1)

    -- this is not allowed: immutable vars, can only be defined once per scope
    foo = True
    foo = False

Local Variables
^^^^^^^^^^^^^^^
You can introduce a new local scope using a let-expression

.. code-block:: haskell

    sum 0 = 0
    sum n = let n' = n - 1  -- n' is only in scope in the in block
            in n + sum n'

    -- multiple lets
    sum 0 = 0
    sum n = let
                n' = n - 1
                sum' = sum n'
            in n + sum'

If you need a var whose scope is an eqn, use ``where``

.. code-block:: haskell

    cmpSquare x y | x > z  = "bigger"
                  | x == z = "equal"
                  | x < z  = "smaller"
        where z = y*y

Types
-----
Lambda-calculus is untyped: for example, ``let FNORD = ONE ZERO``.

In Haskell, every expression either has a type or is **ill-typed** and rejected statically (at compile-time)

Type Annotations
^^^^^^^^^^^^^^^^
You can annotate bindings with types using ``::``

.. code-block:: haskell

    foo :: Bool
    foo = True

    message :: String
    message = if foo
                then "bar"
                else "baz"

    -- word-sized integer
    rating :: Int
    rating = if foo then 10 else 0

    -- arbitrary precision int
    something :: Integer
    something = factorial 100

Functions have arrow types

.. code-block:: haskell

    > :t (\x -> if x then 'a' else 'b')
    (\x -> if x then 'a' else 'b') :: Bool -> Char

    -- annotate function bindings!
    sum :: Int -> Int
    sum 0 = 0
    sum n = n + sum (n - 1)

    -- multiple args
    pair :: String -> (String -> (Bool -> String))
    pair x y b = if b then x else y

    -- same as
    pair :: String -> String -> Bool -> String
    pair x y b = if b then x else y

Lists
^^^^^
A list is:

.. code-block:: haskell

    -- an empty list
    [] -- "nil"

    -- a head element attached to a tail list
    x:xs -- "x cons xs"

    -- examples
    [] -- a list with 0 elements

    1:[] -- [1]

    (:) 1 [] -- for any infix op, (op) is a regular function

    1:(2:(3:(4:[]))) -- [1, 2, 3, 4]

    1:2:3:4:[] -- same as above

    [1,2,3,4] -- guess what this does

``[]`` and ``(:)`` are the list constructors

- ``True`` and ``False`` are ``Bool`` constructors
- ``0, 1, 2`` are... complicated, but basically ``Int`` constructors
- they take 0 args, so we call them values

A list has type ``[A]`` when each of its elements has type ``A``

.. code-block:: haskell

    foo :: [Int]
    foo = [1,2,3]

    bar :: [Char]                   -- = String
    bar = ['h', 'e', 'l', 'l', 'o'] -- = "hello"

    generic :: [t]
    generic = []

Functions on List
"""""""""""""""""

.. code-block:: haskell

    -- range
    upto :: Int -> Int -> [Int]
    upto n m
        | n > m     = []
        | otherwise = n : (upto (n + 1) m)

    -- syntactic sugar:
    [1..7]   -- = [1,2,3,4,5,6,7]
    [1,3..7] -- = [1,3,5,7]

    -- length
    length :: [Int] -> Int
    length []     = 0
    length (_:xs) = 1 + length xs  -- note: a pattern can be applied to other patterns

**Pattern matching** attempts to match values against patterns and, if desired, bind variables to successful values

List Comprehensions
"""""""""""""""""""

.. code-block:: haskell

    [toUpper c | c <- s]
    -- [toUpper(char) for c in s] in Python

    [(i, j) | i <- [1..3],
              j <- [1..i]) -- multiple generators
    -- [(i, j) for i in range(1, 4) for j in range(1, i+1)]

    [(i, j) | i <- [1..3],
              j <- [1..i],
              i + j == 5) -- multiple generators with condition
    -- [(i, j) for i in range(1, 4) for j in range(1, i+1) if i + j == 5]

Pairs
^^^^^

.. code-block:: haskell

    myPair :: (String, Int)
    myPair = ("apple", 3)

``(,)`` is the pair constructor

.. code-block:: haskell

    -- field access
    fruit = fst myPair
    num   = snd myPair

    -- field access using patterns
    isEmpty (x, y) = y == 0

    -- same as
    isEmpty        = \(x, y) -> y == 0
    isEmpty p      = let (x, y) = p in y == 0

What about:

.. code-block:: haskell

    f :: String -> [(String, Int)] -> Int
    f _ [] = 0
    f x ((k,v) : ps)
        | x == k    = v
        | otherwise = f x ps

    -- in Python: f = ((k,v) : ps).get(x, 0)
    -- key-value pair lookups

Tuples
^^^^^^
Go ahead and make n-tuples, they work pretty much as you expect

.. code-block:: haskell

    triple :: (Bool, Int, [Int])
    triple = (True, 1, [1,2,3])

    -- also
    myUnit :: ()
    myUnit = ()
