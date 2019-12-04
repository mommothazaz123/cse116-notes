Type Classes
============

`Slides <https://owenarden.github.io/cse116-fall19/slides/typeclasses.pdf>`_

Quizzes
-------

cse116-plus-type-ind -> E

cse116-ord-ind -> C

cse116-read-ind -> A

Intro
-----

Let's think about overloading operators - ``1 + 1`` and ``1.0 + 1.1`` work slightly differently

This is **ad-hoc overloading** - to compare/add values of multiple types

Note: Haskell has no caste system, so functions are first-class citizens; what class are operators then?

Qualified Types
---------------

.. code-block:: haskell

    :type (+)
    (+) :: (Num a) => a -> a -> a

``+`` takes in any class that is an instance of or implements ``Num`` - Num is a predicate/constraint

A **typeclass** is a collection of operations that must exist for the underlying type.

Eq
^^

.. code-block:: haskell

    class Eq a where
        (==) :: a -> a -> Bool
        (/=) :: a -> a -> Bool

A type a is an instance of ``Eq`` if these operations exist on it.

Creating Instances
------------------

.. code-block:: haskell

    data Unshowable = A | B | C

    instance Eq Unshowable where
        (==) A A = True
        (==) B B = True
        (==) C C = True
        (==) _ _ = False
        (/=) x y = not (x == y)

Automatic Derivation
--------------------

.. code-block:: haskell

    data Showable = A' | B' | C'
        deriving (Eq, Show)

Haskell can automatically generate instances!

Standard Typeclass Hierarchy
----------------------------

.. code-block:: haskell

    class (Eq a, Show a) => Num a where  -- all Nums must derive from Eq and Show
        (+) :: a -> a -> a
        ...

Using Typeclasses
-----------------
Let's build a small lib for environments mapping keys to values:

.. code-block:: haskell

    data Env k v
        = Def v  -- default
        | Bind k v (Env k v)  -- bind k to v, recursive structure
        deriving (Show)

    -- API:
    -- >>> let env0 = add "cat" 10.0 (add "dog" 20.0 (Def 0))

    -- >>> get "cat" env0
    -- 10

    -- >>> get "dog" env0
    -- 20

    -- >>> get "horse" env0
    -- 0

    -- implementation:
    add :: k -> v -> Env k v -> Env k v
    add key val env = Bind key val env

    get :: (Eq k) => k -> Env k v -> v  -- note that k has to derive Eq!
    get key (Def v)          = v
    get key (Bind ek ev env) | k == ek   = ev
                             | otherwise = get key env

What about an optimized version that stores keys in increasing order, to optimize add and get?

1. the types of get and add: ``get :: (Ord k) => k -> Env k v -> v`` need to add Ord
2. the type of Env: move the default so that we don't have to recurse to the end

Explicit Signatures
-------------------
In some cases using typeclasses, explicit signatures are required:

e.g. ``read :: (Read a) => String -> a``, the opposite of ``Show``

We have to do: ``(read "2") :: Int`` or ``(read "2") :: Float``
