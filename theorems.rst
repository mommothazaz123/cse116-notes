Theorems about Programs
=======================

Intro
-----

`Slides <https://owenarden.github.io/cse116-fall19/slides/formal.key.pdf>`_

**Quizzes**

cse116-reduce-ind -> A

cse116-induct-ind -> B

cse116-reduce2-ind -> E

cse116-nano2-ind -> D

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

- expression e makes a step (reduces in one step) to an expression e'

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

Evaluation Order
^^^^^^^^^^^^^^^^
Out of these expressions, only the first is valid:

- ``(1 + 2) + (3 + 4) => 3 + (3 + 4)``
- ``(1 + 2) + (3 + 4) => (1 + 2) + 7``

since expression 1 has a derivation, but expr 2 does not:

.. code-block:: haskell

      [Add] --------------------------------
                        1 + 2 => 3
    [Add-L] --------------------------------
            (1 + 2) + (3 + 4) => 3 + (3 + 4)

    -- but:
      [???] --------------------------------
            (1 + 2) + (3 + 4) => (1 + 2) + 7

Evaluation Relation
^^^^^^^^^^^^^^^^^^^
Like in lambda calc, we define the **multi-step reduction** relation ``e =*> e'``:

``e =*> e'`` iff there exists a sequence of expressions ``e1..en` s.t. ``e1 = e``, ``en = e'``, ``ei => e(i+1)``

Similarly, we can define **evaluation relations** ``e =~> e'``.

Nano1 Thms
^^^^^^^^^^
Let's prove:

- every Nano1 program terminates
- Closed Nano1 programs don't get stuck
- (corollary 1+2): closed nano programs evaluate to a value

using induction!

Induction on terms
""""""""""""""""""

.. code-block:: haskell

    e ::= n | x
            | e1 + e2
            | let x = e1 in e2

To prove ``\forall e.P(e)``, we need to prove:

- BS 1: P(n)
- BS 2: P(x)
- IS 1: P(e1 + e2) assuming P(e1) and P(e2)
- IS 2: P(let x = e1 in e2) assuming P(e1) and P(e2)

Induction on derivations
""""""""""""""""""""""""
The relation ``=>`` is also defined inductively:

- axioms are base cases (``[Add]``, ``[Let]``)
- rules with premises are inductive cases (``[Add-L], [Add-R], [Let-Def]``)

Thm: Termination
^^^^^^^^^^^^^^^^
**Thm 1**: For any expression ``e``, there exists ``e'`` s.t. ``e =~> e'``.

Let's define the size of an expression s.t.:

- size of each expression is positive
- each reduction step strictly decreases the size

.. code-block:: haskell

    size n                  = 1
    size x                  = 1
    size (e1 + e2)          = size e1 + size e2
    size (let x = e1 in e2) = size e1 + size e2

**Lemma 1**: For all ``e``, ``size e > 0``.

- BS 1: ``size n = 1 > 0``.
- BS 2: ``size x = 1 > 0``.
- IS 1: ``size (e1 + e2) = size e1 + size e2 > 0`` because ``size e1 > 0`` and ``size e2 > 0`` by IH.
- IS 2: similar.

**Lemma 2**: For any ``e, e'`` s.t. ``e => e'``, ``size e' < size e``.

Proof: by induction on the derivation of ``e => e'``.

*Base case: [Add]*

- Given: the root of the derivation is ``[Add]: n1 + n2 => n`` where ``n = n1 + n2``.
- To prove: ``size n < size (n1 + n2)``
- ``1 < 2``.

*Inductive case: [Add-L]*

- Given: the root of the derivation is ``[Add-L]``: (defn [Add-L].)
- To prove: ``size (e1' + e2) < size (e1 + e2)``
- IH: ``size e1' < size e1``
- ``size e1' + size e2 < size e1 + size e2`` by addition
- ``size (e1' + e2) < size (e1 + e2)`` by defn of size. QED.

*Base case: [Let]*

- Given: root of the derivation is ``[Let]: let x = v in e2 => e2[x := v]``
- Prove: ``size (e2[x := v]) < size (let x = v in e2)``
- ``size (e2[x := v]) = size e2`` by aux lemma
- ``size (let x = v in e2) = size v + size e2`` by defn
- ``size e2 < size v + size e2`` by lemma 1
- therefore, ``size (e2[x := v]) < size (let x = v in e2)``

Nano2: Adding functions
-----------------------
Let's extend the syntax:

.. code-block:: haskell

    e ::= n | x                          --  expressions
            | e1 + e2
            | let x = e1 in e2
            | \x -> e
            | e1 e2

    v ::= n | (\x -> e)

Operational Semantics
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: haskell

               e1 => e1'
    [App-L] ---------------
            e1 e2 => e1' e2

              e => e'
    [App-R] -----------
            v e => v e'

    [App] (\x -> e) v => e[x := v]


example:

.. code-block:: haskell

    ((\x y -> x + y) 1) (1 + 2)
    => (\y -> 1 + y) (1 + 2)  -- [App-L]|[App]
    => (\y -> 1 + y) 3        -- [App-R]|[Add]
    => 1 + 3                  -- [App]
    => 4                      -- [Add]

Our rules implement call-by-value:

- evaluate the function (to a lambda)
- evaluate the arg (to some value)
- make the call: make a sub of formal to actual in body

the alternative is call-by-name:

- do not evaluate the argument before making the call
- let's modify the rules to make it call by name!

**modified call-by-name**:

.. code-block:: haskell

               e1 => e1'
    [App-L] ---------------
            e1 e2 => e1' e2

    [App] (\x -> e1) e2 => e1[x := e2]

Thms about Nano2
^^^^^^^^^^^^^^^^

- not every program will terminate! think of the omega term
- programs can get stuck! what about ``1 2``?