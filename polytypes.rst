Polymorphism & Type Inference
=============================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/types.key.pdf>`_

**Quizzes**

cse116-nanotype-ind -> D1

cse116-typed-ind -> B

Type System
-----------
A type system defines what types an expression can have

To define a type system, we need to define:

- the syntax of types: what do types look like?
- the static semantics of our language (i.e. the typing rules): assign types to expressions

Syntax of Types
^^^^^^^^^^^^^^^

.. code-block:: haskell

    T ::= Int       -- integers
        | T1 -> T2  -- function types

Now, we define a typing relation ``e :: T`` ("e has type T"), inductively thru typing rules:

.. code-block:: haskell

    [T-Num] n :: Int

            e1 :: Int    e2 :: Int  -- premises
    [T-Add] ----------------------
                e1 + e2 :: Int      -- conclusions

    [T-Var] x :: ???

Type Environment
^^^^^^^^^^^^^^^^
An expression has a type in a given type environment (or context), which maps all its free variables to their types:

.. code-block:: haskell

    G = x1:T1, x2:T2, ..., xn:Tn

    -- now, our typing relation should include G:
    G |- e :: T  -- e has type T in G

Typing Rules
^^^^^^^^^^^^
An expression e has type T if we can derive ``G |- e :: T`` using these rules

An expression e is well-typed in G if we can derive ``G |- e :: T`` for some type T

.. code-block:: haskell

    -- typing rules using G
    [T-Num] G |- n :: Int

            G |- e1 :: Int    G |- e2 :: Int
    [T-Add] --------------------------------
                G |- e1 + e2 :: Int

    [T-Var] G |- x :: T             if x:T in G

               G,x:T1 |- e :: T2
    [T-Abs] ------------------------
            G |- \x -> e :: T1 -> T2

            G |- e1 :: T1 -> T2    G |- e2 :: T1
    [T-App] ------------------------------------  -- modus ponens!
                    G |- e1 e2 :: T2

            G |- e1 :: T1    G,x:T1 |- e2 :: T2
    [T-Let] -----------------------------------
               G |- let x = e1 in e2 :: T2

.. note::

    examples:

    .. code-block:: haskell

        -- 1
        [] |- (\x -> x) 2 :: Int

        [T-Var]  -------------------
                 [x:Int] |- x :: Int
        [T-Abs]  -------------------              --------------  [T-Num]
                 [] |- \x -> x :: Int -> Int      [] |- 2 :: Int
        [T-App]  -----------------------------------------------
                 [] |- (\x -> x) 2 :: Int

        -- 2
        [] |- let x = 1 in x + 2 :: Int   
        
        [T-Var] -----------------   -----------------[T-Num]
                x:Int |- x :: Int   x:Int |- 2 :: Int
        [T-Num] --------------   ------------------------------------[T-Add]
                [] |- 1 :: Int   x:Int |- x + 2 :: Int
        [T-Let] -----------------------------------
                [] |- let x = 1 in x + 2 :: Int

    ``[] |- (\x -> x x) :: T`` is underivable, because T has to be equal to ``T -> T``


According to these rules, an expression can have zero, one, or many types.

e.g. ``1 2`` has no types, ``1`` has 1 type, ``\x -> x`` has many types.

One problem with this system: there's no generics.