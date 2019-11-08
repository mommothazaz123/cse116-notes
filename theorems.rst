Theorems about Programs
=======================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/formal.key.pdf>`_

**Quizzes**

cse116-reduce-ind -> A

.. code-block:: haskell

    [Add]     --------------------------------------------------------
                                     1 + 2 => 3
    [Let-Def] --------------------------------------------------------
              (let x = 1 + 2 in 4 + 5 + x) => (let x = 3 in 4 + 5 + x)

Formalizing Nano
----------------

We want to be able to guarantee properties about programs, such as:

- evaluation is deterministic
- all programs terminate
- certain programs never fail at runtime
- etc.

To prove theorems about programs we first need to define formally

- their syntax (what programs look like)
- their semantics (what it means to run a program)

Let’s start with Nano1 (Nano w/o functions) and prove some stuff!

Nano1: Syntax
^^^^^^^^^^^^^

.. code-block:: haskell

    e ::= n | x                          --  expressions
            | e1 + e2
            | let x = e1 in e2

    v ::= n                                  --  values

where n ∈ ℕ, x ∈ Var

Nano1: Operational Semantics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Operational semantics defines how to execute a program step by step

Let’s define a step relation (reduction relation) e  =>  e'

- “expression e makes a step (reduces in one step) to an expression e'

We define the step relation inductively through a set of rules:

.. code-block:: haskell

                   e1 => e1'           --  premise
    [Add-L]   -------------------
              e1 + e2 => e1' + e2      --  conclusion

                   e2 => e2'
    [Add-R]   -------------------
              n1 + e2 => n1 + e2'

    [Add]     n1 + n2 => n              where  n  ==  n1  +  n2

                            e1 => e1'
    [Let-Def] -------------------------------------
              let x = e1 in e2 => let x = e1' in e2

    [Let]     let x = v in e2 => e2[x := v]

and we can define ``e[x := v]`` as:

.. code-block:: haskell

    x[x := v]                  = v
    y[x := v]                  = y
    n[x := v]                  = n
    (e1 + e2)[x := v]          = e1[x := v] + e2 [x := v]
    (let x = e1 in e2)[x := v] = let x = e1[x := v] in e2
    (let y = e1 in e2)[x := v] = let x = e1[x := v] in e2[x := v]

A reduction is valid if we can build its derivation by stacking the rules:

.. code-block:: haskell

      [Add] --------------------
                  1 + 2 => 3
    [Add-L] --------------------
            (1 + 2) + 5 => 3 + 5

Note: we don't have reduction rules for *n* or *x*, since both these expressions cannot be further reduced (normal).

However, *x* is not a value, and if the final result is that, it's a runtime error (**stuck**)
