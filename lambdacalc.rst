Lambda Calculus
===============

Review: `9/26 - Lambda Calculus <https://docs.google.com/document/d/1VjnJagBNYv_BmEGZgROulERCcNDfIkiv20c7mZr_gns/edit#heading=h.m1n86y93ovps>`_

`Slides <https://owenarden.github.io/cse116-fall19/slides/lambda.key.pdf>`_

.. data:: occurrence

    an appearance of a variable in an expression (binding does not count)

Quizzes
-------
tiny.cc/

cse116-lambda-ind -> A

cse116-scope-ind -> C

cse116-beta1-ind -> D

cse116-beta2-ind -> A

cse116-norm-ind -> C

cse116-church-ind -> A

cse116-add-ind -> A

cse116-mult-ind -> B

cse116-sum-ind -> NO


Reductions
----------
.. data:: alpha-reduction

    ``\x -> e =a> \y -> e[x := y]``
        ``| where not (y in FV(e))``


.. data:: beta-reduction

    ``(\x -> e1) e2 =b> e1[x := e2]``

    "Replace all **free** occurrences of *x* in *e1* with *e2*."

.. code-block:: haskell

    x[x := e] = e
    y[x := e] = y
    (e1 e2)[x := e] = (e1[x := e]) (e2[x := e])
    (\x -> e1)[x := e] = \x -> e1                   -- since x in e1 is bound
    (\y -> e1)[x := e]
      | not (y in FV(e)) = \y -> e1[x := e]
      | otherwise undefined

Normal Forms
------------

A **redex** is a lambda-term of the form ``(\x -> e1) e2`` (i.e. can be beta-reduced).

A lambda-term is in **normal form** if it contains no redexes (i.e. cannot be beta-reduced).

Semantics: Evaluation
---------------------

A lambda-term *e* evaluates to *e'* if:
1. There is a sequence of stops ``e =?> e_1 =?> ... =?> e'``

Examples
^^^^^^^^
.. code-block:: haskell

    (\x -> x) apple
        =b> apple

    (\f -> f (\x -> x)) (\x -> x)
        =b> (\x -> x) (\x -> x)
        =b> \x -> x

    (\x -> x x) (\x -> x)
        =b> (\x -> x) (\x -> x)
        =b> \x -> x

Elsa Shortcuts
^^^^^^^^^^^^^^

Named lambda-terms
""""""""""""""""""
``let ID = \x -> x``

To substitute a name with its defn, use a =d> step

.. code-block:: haskell

    ID apple
        =d> \x -> x apple
        =b> apple

Evaluation
""""""""""
``e1 =*> e2`` - e1 reduces to e2 in 0 or more steps, where each step is in =a>, =b>, =d>

``e1 =~> e2`` - e1 evaluates to e2 (i.e. final output)

Non-Terminating Evaluation
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: haskell

    (\x -> x x) (\x -> x x)
        =b> (\x -> x x) (\x -> x x)

Programs can loop and never reduce to normal form!

This is called the omega-term.

What if we pass omega to another function?

.. code-block:: haskell

    let OMEGA = (\x -> x x) (\x -> x x)
    (\x -> \y -> y) OMEGA

Lambda Calculus: Booleans
-------------------------
How do we encode T/F as a func?

With booleans, we make a binary choice (e.g. ``if b then e1 else e2``)

We need to define three functions:

.. code-block:: haskell

    let TRUE = \x y -> x
    let FALSE = \x y -> y
    let ITE = \b x y -> b x y

such that

.. code-block:: haskell

    ITE TRUE apple banana =~> apple
    ITE FALSE apple banana =~> banana

.. code-block:: haskell

    eval ite_true:
        ITE TRUE e1 e2
        =d> (\b x y -> b x y)     TRUE e1 e2
        =b>   (\x y -> TRUE x y)  e1 e2
        =b>     (\y -> TRUE e1 y) e2
        =b>            TRUE e1 e2
        =d> (\x y -> x) e1 e2
        =b>   (\y -> e1) e2
        =b> e1

    eval ite_false:
        ITE FALSE e1 e2
        =d> (\b x y -> b x y)      FALSE e1 e2
        =b>   (\x y -> FALSE x y)  e1 e2
        =b>     (\y -> FALSE e1 y) e2
        =b>            FALSE e1 e2
        =d> (\x y -> y) e1 e2
        =b>   (\y -> y) e2
        =b> e2

Now we can define other boolean operators:

.. code-block:: haskell

    let NOT = \b     -> ITE b FALSE TRUE
    let AND = \b1 b2 -> ITE b1 b2 FALSE
    let OR  = \b1 b2 -> ITE b1 TRUE b2

(``ITE`` is redundant, so it can be removed from these defns)

Lambda Calculus: Records
------------------------
- Start with records w/ 2 fields (pairs)
- What do we want to do?
    - Pack two items into a pair
    - Get first
    - Get second

API
^^^
.. code-block:: haskell

    let PAIR = \x y -> (\b -> ITE b x y)
        -- a function that returns a function
            -- that takes a boolean asking which item you want
    let FST  = \p -> p TRUE
    let SND  = \p -> p FALSE

such that

.. code-block:: haskell

    FST (PAIR apple banana) =~> apple
    SND (PAIR apple banana) =~> banana

Triples
^^^^^^^

.. code-block:: haskell

    let TRIPLE = \x y z -> PAIR x (PAIR y z)
    let FST3   = \t -> FST t
    let SND3   = \t -> FST (SND t)
    let TRD3   = \t -> SND (SND t)

Lambda Calculus: Numbers
------------------------

- What about natural numbers [0..]?
- Counters, arithmetic, comparisons
- +, -, \*, ==, <=, etc

We need to define:

- a family of numerals ``ZERO, ONE, TWO``, etc
- arithmetic functions ``INC, DEC, ADD, SUB, MULT``
- comparisons ``IS_ZERO, EQ``

Implementation
^^^^^^^^^^^^^^
**Church numerals**: A number N is encoded as a combinator that calls a function on an argument N times

.. code-block:: haskell

    let ZERO  = \f x -> x
    let ONE   = \f x -> f x
    let TWO   = \f x -> f (f x)
    let THREE = \f x -> f (f (f x))
    ...etc

Increment
"""""""""

.. code-block:: haskell

    -- call `f` on `x` one more time than `n` does
    let INC = \n -> (\f x -> f (n f x))

    -- ex
    INC ZERO
        =d> (\n f x -> f (n f x)) ZERO
        =b> \f x -> f (ZERO f x)
        =*> \f x -> f x
        =d> ONE

Add
"""

.. code-block:: haskell

    let ADD = \n m -> n INC m
    -- n is a function that takes a function and number
    -- i.e. apply INC n times to m

    -- ex
    eval add_one_zero:
        ADD ONE ZERO
            =d> (\n m -> n INC m) ONE ZERO
            =b> (\m -> ONE INC m) ZERO
            =b> ONE INC ZERO
            =d> (\f x -> f x) INC ZERO
            =b> INC ZERO
            =*> ONE

    eval add_two_one:
        ADD TWO ONE
            =d> (\n m -> n INC m) TWO ONE
            =b> (\m -> TWO INC m) ONE
            =b> TWO INC ONE
            =d> (\f x -> f (f x)) INC ONE
            =b> INC (INC ONE)
            =*> THREE

Mult
""""

.. code-block:: haskell

    let MULT = \n m -> n (ADD m) ZERO
    -- ADD m returns a function
    -- so we call ADD m on ZERO n times
    -- similar to python partials

    -- ex
    eval two_times_one:
        MULT TWO ONE
            =d> (\n m -> n (ADD m) ZERO) TWO ONE
            =b> (\m -> TWO (ADD m) ZERO) ONE
            =b> TWO (ADD ONE) ZERO
            =~> ADD ONE (ADD ONE ZERO)
            =~> TWO

Lambda Calculus: Recursion
--------------------------
Ex. I want to write a number that sums up natural numbers to n.

- ``\n -> ... -- = 1 + 2 + ... + n``

Step 1: Pass in the function to call recursively

.. code-block:: haskell

    let STEP =
        \rec ->
            \n -> ITE (ISZ n)
                ZERO
                (ADD n (rec (DEC n)))

Step 2: Do something to ``STEP`` so that the function passed as ``rec`` becomes:

``\n -> ITE (ISZ n) ZERO (ADD n (rec (DEC n)))``

Fixpoint Combinator
"""""""""""""""""""
.. note::
    Wanted: a combinator ``FIX`` s.t. ``FIX STEP`` calls ``STEP`` with itself as the first argument

    .. code-block:: haskell

        FIX STEP
            =*> STEP (FIX STEP)

    .. note::
        It's important that ``STEP`` has some base case in it, or else you end up with ``STEP (STEP (STEP (STEP...)))``

    then, ``let SUM = FIX STEP``, so ``SUM =*> STEP SUM``

    .. code-block:: haskell

        eval sum_one:
            SUM ONE
                =*> STEP SUM ONE
                =d> (\rec n -> ITE (ISZ n) ZERO (ADD n (rec (DEC n)))) SUM ONE
                =b> (\n -> ITE (ISZ n) ZERO (ADD n (SUM (DEC n)))) ONE
                =b> ITE (ISZ ONE) ZERO (ADD ONE (SUM (DEC ONE)))
                =*> ITE FALSE ZERO (ADD ONE (STEP SUM ZERO))
                =*> ADD ONE (SUM ZERO)
                =*> ADD ONE (STEP SUM ZERO)
                =d> ADD ONE ((\rec n -> ITE (ISZ n) ZERO (ADD n (rec (DEC n)))) SUM ZERO)
                =b> ADD ONE ((\n -> ITE (ISZ n) ZERO (ADD n (SUM (DEC n)))) ZERO)
                =b> ADD ONE (ITE (ISZ ZERO) ZERO (ADD ZERO (SUM (DEC ZERO))))
                =*> ADD ONE (ITE TRUE ZERO (ADD ZERO (SUM (DEC ZERO))))
                =*> ADD ONE ZERO
                =~> ONE

So how do we define ``FIX``?

- Let's look back at omega:
    - ``(\x -> x x) (\x -> x x) =b> (\x -> x x) (\x -> x x)``
- We need something similar but with control
- Thus, the Y combinator (or fixpoint)

.. code-block:: haskell

    let FIX = \stp -> (\x -> stp (x x)) (\x -> stp (x x))

    eval fix_step:
        FIX STEP
        =d> (\stp -> (\x -> stp (x x)) (\x -> stp (x x))) STEP
        =b> (\x -> STEP (x x)) (\x -> STEP (x x))
        =b> STEP ((\x -> STEP (x x)) (\x -> STEP (x x)))
        =d> STEP (FIX STEP)

.. note::
    Example: ``MULT`` using recursion

    .. code-block:: haskell

        -- if we can use recursion by name:
        let MULT x y =
            ITE (ISZ y)
                ZERO
                ADD x (MULT x (DECR y))

        -- replace the self ref with a passed func
        let MULT1 f x y =
            ITE (ISZ y)
                ZERO
                ADD x (f x (DECR y))

        -- and use fixpt
        let MULT = FIX MULT1

        -- therefore, generally
        let FUNC0 = \f n -> ... f (DECR n)
        let FUNC = FIX FUNC0
