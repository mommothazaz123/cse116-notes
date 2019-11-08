Datatypes and Recursion
=======================

`Slides <https://owenarden.github.io/cse116-fall19/slides/adt-rec.key.pdf>`_

**Quizzes**

cse116-para-ind -> C
cse116-adt-ind -> D
cse116-case-ind -> B
cse116-case2-ind -> D
cse116-rectype-ind -> E
cse116-tree-ind -> C
cse116-leaves-ind -> D
cse116-tail-ind -> NO

Representing complex data
-------------------------
- base/primitive types: int, float, bool, etc
- ways to build up types: functions, tuples, lists

**Algebraic Data Types**: a technique to build data types from these

.. note::

    Tuples can do the job, but there are two problems:

    - verbose and unreadable
    - no type checking (unsafe)

    .. code-block:: haskell

        type Date = (Int, Int, Int)
        type Time = (Int, Int, Int)

        deadDate :: Date
        deadDate = (2, 4, 2019)

        deadTime :: Time
        deadTime = (11, 59, 59)

        -- example: extend
        extension :: Date -> Date
        extension = ...

        -- however, you can do
        extension deadTime -- which should error!

Solution: construct *datatypes*

.. code-block:: haskell

    data Date = Date Int Int Int
    data Time = Time Int Int Int
    -- constructor ^  ^ param types

    deadDate :: Date
    deadDate = Date 2 4 2019

    deadTime :: Time
    deadTime = Time 11 59 59

Building Data Types
-------------------
1. Product types (each-of): a value of ``T`` contains a value of ``T1`` and a value of ``T2``
2. Sum types (one-of): A value of ``T`` contains a value of ``T1`` *or* a value of ``T2``
3. Recursive types: A value of ``T`` contains subvalues of type ``T``

Product Types
^^^^^^^^^^^^^
You can name the constructor params:

.. code-block:: haskell

    data Date = Date {
        month :: Int,
        day :: Int,
        year :: Int
    }

    deadDate = Date 2 4 2019
    deadMonth = month deadDate
    -- field name is func that accesses date

Sum Types
^^^^^^^^^
e.g. a type for ``Paragraph`` that is one of the three options

.. code-block:: haskell

    data Paragraph =
          Text String
        | Heading Int String
        | List Bool [String]

Recursive Types
^^^^^^^^^^^^^^^
See :ref:`recursive-types`

Constructing Datatypes
----------------------

.. code-block:: haskell

    data T =
          C1 T11 .. T1k
| C2 T21 .. T2l
| ..
| Cn Tn1 .. Tnm

``T`` is the **datatype**

``C1 .. Cn`` are the **constructors**

A **value** of type ``T`` is

- either ``C1 v1 .. vk`` with ``vi :: T1i``
- or ``C2 v1 .. vl`` with ``vi :: T2i``
- or ...
- or ``Cn v1 .. vm`` with ``vi :: Tni``

Writing Functions
-----------------
e.g. how to write a function to convert nanoMD to HTML?

Pattern Matching
^^^^^^^^^^^^^^^^
match on the constructor

.. code-block:: haskell

    html :: Paragraph -> String
    html (Text str) = ...
    html (Heading lvl str) = ...
    html (List ord items) = ...

But, there are dangers:

.. code-block:: haskell

    -- example: missing a type
    html :: Paragraph -> String
    html (Text str) = ...
    html (List ord items) = ...

    html (Heading 1 "Introduction") -- runtime error!

You can also pattern match inside the program:

.. code-block:: haskell

    html :: Paragraph -> String
    html p =
        case p of
            Text str -> ...
            Heading lvl str -> ...
            List ord items -> ...

Case
^^^^

.. code-block:: haskell

    case e of
        pattern1 -> e1
        pattern2 -> e2
        ...
        patternN -> eN

has type ``T`` if:

- each ``e1..eN`` has type ``T``
- ``e`` has some type ``D``
- each ``pattern1..patternN`` is a valid pattern for ``D``

Recursive Types
---------------

Let's define natural numbers.

.. code-block:: haskell

    data Nat = Zero       -- base constructor
               | Succ Nat -- inductive constructor

    Zero      -- 0
    Succ Zero -- 1

A ``Nat`` value is a box named ``Zero`` or a box labeled ``Succ`` with another ``Nat`` in it

Using as Parameter
^^^^^^^^^^^^^^^^^^

.. code-block:: haskell

    toInt :: Nat -> Int
    toInt Zero     = 0           -- base case
    toInt (Succ n) = 1 + toInt n -- inductive case

Using as Result
^^^^^^^^^^^^^^^

.. code-block:: haskell

    fromInt :: Int -> Nat
    fromInt n
        | n <= 0    = Zero
        | otherwise = Succ (fromInt (n - 1))

    -- and operations
    add :: Nat -> Nat -> Nat
    add Zero     m = m
    add (Succ n) m = Succ (add n m)

    sub :: Nat -> Nat -> Nat
    sub n        Zero     = n
    sub Zero     _        = Zero
    sub (Succ n) (Succ m) = sub n m

Lists
^^^^^
Lists aren't built in!

.. code-block:: haskell

    data List = Nil
        | Cons Int List

    [1, 2, 3] == Cons 1 (Cons 2 (Cons 3 Nil))

Ex. appending two lists

.. code-block:: haskell

    append :: List -> List -> List
    append [] ys     = ys
    append (x:xs) ys = x:(append xs ys)

    append2 :: List -> List -> List
    append2 xs []     = xs
    append2 xs (y:ys) = append xs:y ys

Trees
^^^^^
Think of lists as unary trees with elements stored in the nodes.
What about binary trees?

.. code-block:: haskell

    data Tree = Leaf | Node Int Tree Tree  -- leaves don't store data!

    t1234 = Node 1
                (Node 2 (Node 3 Leaf Leaf) Leaf)
                (Node 4 Leaf Leaf)

    1 - 2 - 3 - ()
      |   |   \ ()
      |   \ ()
      \ 4 - ()
          \ ()

Functions on Trees
""""""""""""""""""

.. code-block:: haskell

    depth :: Tree -> Int
    depth Leaf = 0
    depth (Node _ l r) = 1 + max (depth l) (depth r)

Ex: Calculator
""""""""""""""
Let's implement an arithmetic calculator to eval things like 4.0 + 2.0, 3 - 9, (4.0 + 2.9) * (1.0 + 2.2)

.. code-block:: haskell

    data Expr = Val Float
                | Add Expr Expr
                | Sub Expr Expr
                | Mul Expr Expr

    -- evaluate!
    eval :: Expr -> Float
    eval (Num f)     = f
    eval (Add e1 e2) = eval e1 + eval e2
    eval (Sub e1 e2) = eval e1 - eval e2
    eval (Mul e1 e2) = eval e1 * eval e2

Tail Recursion
--------------
Whatever the recursive call returns will be what the expression returns.
No computations are allowed on recursively returned values.


.. code-block:: haskell

    -- tail recursive factorial!
    facTR :: Int -> Int
    facTR n = loop 1 n
        where
            loop :: Int -> Int -> Int
            loop acc n
                | n <= 1    = acc
                | otherwise = loop (acc * n) (n - 1)

    --      <facTR 4>
    --     <<loop 1 4>>
    --    <<<loop 4 3>>>
    --   <<<<loop 12 2>>>>
    --  <<<<<loop 24 1>>>>>
    -- <<<<<<24>>>>>>
