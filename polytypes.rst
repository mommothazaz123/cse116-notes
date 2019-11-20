Polymorphism & Type Inference
=============================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/types.key.pdf>`_

**Quizzes**

cse116-nanotype-ind -> D1

cse116-typed-ind -> B

cse116-subst-ind -> B

cse116-unify-ind -> C, D, E

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

Polymorphic Types
-----------------
We can formalize a type ``a -> a`` as a polymorphic type: ``forall a . a -> a``

- where ``a`` is a bound type variable
- also called a type scheme
- haskell has polymorphic types, but forall isn't usually required

We can instantiate this scheme into different types by replacing ``a`` in the body with some type, e.g.
instantiating with ``Int`` yields ``Int -> Int``.

.. note::
    Similar to lambda expression at type level

With polymorphic types, we can derive ``e :: Int -> Int`` where ``e`` is

.. code-block:: haskell

    let id = \x -> x in
        let y = id 5 in
            id (\z -> z + y)

Inference works as follows:

1. When we have to pick a type T for x, we pick a fresh type variable a
2. So the type of ``\x -> x`` comes out as ``a -> a``
3. We can generalize this type to ``forall a . a -> a``
4. When we apply id the first time, we instantiate this polymorphic type with ``Int``
5. When we apply id the second time, we instantiate this polymorphic type with ``Int ->Int``

Type System 3
^^^^^^^^^^^^^
Types:

.. code-block:: haskell

    -- Mono-types
    T ::= Int
        | T1 -> T2
        | a             -- type variables

    -- Poly-types
    S ::= T             -- mono
        | forall a . S  -- polymorphic

    -- where a ∈ TVar, T ∈ Type, S ∈ Poly

Type Environment
""""""""""""""""

The type environment now maps variables to poly-types: ``G : Var -> Poly``

- example, ``G = [z: Int, id: forall a . a -> a]``

Type Substitutions
""""""""""""""""""

We need a mechanism for replacing all type variables in a type with another type:

A type substitution is a finite map from type variables to types: ``U : TVar -> Type``

- example: ``U1 = [a / Int, b / (c -> c)]``

To apply a substitution U to a type T means replace all type vars in T with whatever they are mapped to in U

- example 1: ``U1 (a -> a) = Int -> Int``
- example 2: ``U1 Int = Int``

Typing Rules
""""""""""""
We need to change the typing rules so that:

.. code-block:: haskell

    -- 1. variables and their definitions can have polymorphic types
    [T-Var] G |- x :: S          if x:S in G

            G |- e1 :: S   G, x:S |- e2 :: T
    [T-Let] ------------------------------------
               G |- let x = e1 in e2 :: T

    -- 2. we can instantiate a type scheme into a type
             G |- e :: forall a . S
    [T-Inst] ----------------------
              G |- e :: [a / T] S

    -- 3. we can generalize a type with free type variables into a type scheme
                 G |- e :: S
    [T-Gen] ---------------------- if not (a in FTV(G))  -- FTV = Free Type Variables
            G |- e :: forall a . S

    -- the rest of the rules are the same:
    [T-Num] G |- n :: Int

            G |- e1 :: Int    G |- e2 :: Int
    [T-Add] --------------------------------
                G |- e1 + e2 :: Int

               G,x:T1 |- e :: T2
    [T-Abs] ------------------------
            G |- \x -> e :: T1 -> T2

            G |- e1 :: T1 -> T2    G |- e2 :: T1
    [T-App] ------------------------------------  -- modus ponens!
                    G |- e1 e2 :: T2

Examples
""""""""

.. code-block:: haskell

    -- derive: [] |- \x -> x :: forall a . a -> a

    [T-Var] ---------------
            [x:a] |- x :: a
    [T-Abs] -----------------------
            [] |- \x -> x :: a -> a
    [T-Gen] ----------------------------------  not (a in FTV([]))
            [] |- \x -> x :: forall a . a -> a

    -- derive: [x:a] |- x :: forall a . a
    -- not derivable, since a is not in FTV([x:a])

    -- derive: G1 |- id 5 :: Int where G1 = [id : (forall a . a -> a)]

    [T-Var] -----------------------------
            G1 |- id :: forall a . a -> a
    [T-Inst]----------------------         -------------- [T-Num]
            G1 |- id :: Int -> Int         G1 |- 5 :: Int
    [T-App] ---------------------------------------------
            G1 |- id 5 :: Int

    -- see slides page 12 for example 3

Representing Types
^^^^^^^^^^^^^^^^^^
The eventual goal is to create a function ``infer``, which:

- given a context G and an expression e,
- returns a type T s.t. ``G |- e :: T``
- or reports a type error

.. code-block:: haskell

    data Type = TInt     -- int
        | Type :=> Type  -- T1 -> T2
        | Var String     -- a, b, c

    data Poly = Mono Type
        | Forall TVar Poly

    type TVar = String
    type TEnv = [(Id, Poly)]  -- type environment
    type Subst = [(String, Type)] -- type sub

**Main idea**: let's implement infer like this:

1. Depending on the kind of expression, find the typing rule that applies to it
2. If the rule has premises, recursively call ``infer`` to obtain the types of subexpressions
3. Combine the types of subexpressions according to the conclusion of the rule
4. If no rule applies, report a type error

The problem is, some of our typing rules are nondeterministic (see slides pg. 13)

1. guessing type
2. guessing when to generalize

solution:

1. whenever we need to guess a type, don't. just return a fresh type variable
2. whenever a rule imposes a constraint on a type, try to find the right substitution for the free type vars to satisfy the constraint (unification)

Unification
^^^^^^^^^^^
The unification problem: given two types T1 and T2, find a type substitution U s.t. ``U T1 = U T2``.

Such a substitution is called a unifier of T1 and T2.

e.g.:

1. The unifier of ``a`` and ``Int`` is ``[a/Int]``
2. ``a -> a`` and ``Int -> Int`` is ``[a/Int]``
3. ``a -> Int`` and ``Int -> b`` is ``[a/Int, b/Int]``
4. ``Int`` and ``Int`` is ``[]``
5. ``a`` and ``a`` is ``[]``
6. ``Int`` and ``Int -> Int`` is invalid
7. ``Int`` and ``a -> a`` is invalid
8. ``a`` and ``a -> a`` is invalid
9. ``b`` and ``a -> a`` is ``[b/a -> a]``
