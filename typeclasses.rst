Type Classes
============

Quizzes
-------

cse116-plus-type-ind -> E

cse116-ord-ind -> C

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
