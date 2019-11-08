Environments & Closures
=======================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/env.key.pdf>`_

**Quizzes**

cse116-vars-ind -> E

cse116-free-ind -> B

cse116-cscope-ind -> B

cse116-env-ind -> A

cse116-enveval-ind -> D

cse116-enveval2-ind -> C

The Nano Language
-----------------
Features of Nano:

1. Arithmetic expressions
^^^^^^^^^^^^^^^^^^^^^^^^^

.. _impl1:

Evaluator 1
"""""""""""

.. code-block:: haskell

    e ::= n
        | e1 + e2
        | e1 - e2
        | e1 * e2

    -- haskell representation:
    data Binop = Add | Sub | Mul
    data Expr = Num Int
                | Bin Binop Expr Expr

    -- evaluator:
    eval :: Expr -> Int
    eval (Num n)         = n
    eval (Bin Add e1 e2) = eval e1 + eval e2
    eval (Bin Sub e1 e2) = eval e1 - eval e2
    eval (Bin Mul e1 e2) = eval e1 * eval e2

2. Variables and let-bindings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: haskell

    e ::= n | x
        | e1 + e2 | e1 - e2 | e1 * e2
        | let x = e1 in e2

    -- haskell representation:
    type Id = String

    data Expr = Num Int                -- number
                | Var Id               -- variable
                | Bin Binop Expr Expr  -- binary op
                | Let Id Expr Expr     -- let expr

thus, expressions must be evaluated in :ref:`environments`

.. _impl2:

Evaluator 2
"""""""""""


Previous implementation: :ref:`impl1`

.. code-block:: haskell

    type Value = Int
    data Env = ...

    -- add new id/value to env
    add :: Id -> Value -> Env -> Env

    -- lookup id in env
    lookup :: Id -> Env -> Value

    -- evaluator:
    eval :: Env -> Expr -> Value
    eval env (Num n)           = n
    eval env (Var x)           = lookup x env
    eval env (Bin op e1 e2)    = f v1 v2
        where
            v1 = eval env e1
            v2 = eval env e2
            f  = case op of
                Add -> (+)
                Sub -> (-)
                Mul -> (*)
    eval env (Let x e1 e2)     = eval env' e2
        where
            v    = eval env e1
            env' = add x v env

Runtime Errors
""""""""""""""
Lookups can fail when a var is not bound!

How do we ensure that it doesn't raise a runtime error?

In ``eval env e``, *env* must contain bindings for all free vars of *e*. Evaluation only succeeds when all expressions are closed.


3. Functions
^^^^^^^^^^^^
Let's add lambda abstractions and function application!

.. code-block:: haskell

    e ::= n | x
        | e1 + e2 | e1 - e2 | e1 * e2
        | let x = e1 in e2
        | \x -> e  -- abstraction
        | e1 e2    -- application

    -- haskell representation:
    data Expr = Num Int                -- number
                | Var Id               -- variable
                | Bin Binop Expr Expr  -- binary op
                | Let Id Expr Expr     -- let expr
                | Lam Id Expr          -- abstraction
                | App Expr Expr        -- application

.. note::

    Now, let's try to evaluate something...

    .. code-block:: haskell

        eval [] {let c = 42 in let cTimes = \x -> c * x in cTimes 2}
        => eval [c:42] {let cTimes = \x -> c * x in cTimes 2}
        => eval [cTimes:???, c:42] {cTimes 2}

    How do we represent lambdas as a value? Let's try ``data Value = VNum Int | VLam Id Expr`` and evaluate...

    .. code-block:: haskell

        eval [] {let c = 42 in let cTimes = \x -> c * x in cTimes 2}
        => eval [c:42] {let cTimes = \x -> c * x in cTimes 2}
        => eval [cTimes:(\x -> c * x), c:42] {cTimes 2}
        => eval [cTimes:(\x -> c * x), c:42] {(\x -> c * x) 2}
        => eval [x:2, cTimes:(\x -> c * x), c:42] {x * c}
        => 42 * 2
        => 84

    But what if *c* is redefined before *cTimes* is used?

    The problem that this brings up is **static v. dynamic** scoping; static scoping = most recent binding in text,
    whereas dynamic = most recent binding in execution

How do we implement lexical scoping? See :ref:`closures`

Now let's update our evaluator! Previous implementation: :ref:`impl2`

.. _impl3:

Evaluator 3
"""""""""""

.. code-block:: haskell

    data Value = VNum Int    -- new!
        | VClos Env Id Expr  -- env + formal + body

    eval :: Env -> Expr -> Value
    eval env (Num n)           = VNum n  -- we must wrap in VNum now!
    eval env (Var x)           = lookup x env
    eval env (Bin op e1 e2)    = VNum (f v1 v2)
        where
            (VNum v1) = eval env e1
            (VNum v2) = eval env e2
            f  = case op of
                Add -> (+)
                Sub -> (-)
                Mul -> (*)
    eval env (Let x e1 e2)     = eval env' e2
        where
            v    = eval env e1
            env' = add x v env
    -- new!
    eval env (Lam x body)      = VClos env x body
    eval env (App fun arg)     = eval bodyEnv body
        where
            (VClos closEnv x body) = eval env fun  -- eval function to closure
            vArg                   = eval env arg  -- eval argument
            bodyEnv                = add x vArg closEnv

But note: this evaluator doesn't cover recursion!

4. Recursion
^^^^^^^^^^^^

We have to do this in homework, yay! See hw4.

.. _environments:

Environments
------------
an environment maps all free vars to values

.. code-block:: haskell

    x * y
    =[x:17, y:2]=> 34

    x * y
    =[x:17]=> Error: unbound var y

    x * (let y = 2 in y)
    =[x:17]=> 34

To evaluate ``let x = e1 in e2`` in ``env``:

- evaluate ``e2`` in an extended env ``env + [x:v]``
- where ``v = eval e1``

.. _closures:

Closures
--------
Closure = lambda abstraction (formal + body) + environment at function definition

a closure environment must save all free variables of a function defn!

.. code-block:: haskell

    data Value = VNum Int
        | VClos Env Id Expr  -- env + formal + body

    -- our syntax:
    -- binding:<env, lambda>

    -- now, eval:
    eval [] {let c = 42 in let cTimes = \x -> c * x in let c = 5 in cTimes 2}
        => eval [c:42] {let cTimes = \x -> c * x in let c = 5 in cTimes 2}
        => eval [cTimes:<[c:42], \x -> c * x>, c:42] {let c = 5 in cTimes 2}
        => eval [c:5, cTimes:<[c:42], \x -> c * x>, c:42] {cTimes 2}
        => eval [c:5, cTimes:<[c:42], \x -> c * x>, c:42] {<[c:42], \x -> c * x> 2}
        -- restore env to the one inside the closure, then bind 2 to x:
        => eval [x:2, c:42] {c * x}
        => 42 * 2
        => 84
