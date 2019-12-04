Monads
======

Abstracting Code Patterns
-------------------------

Recall: the :ref:`map` HOF works on lists

What if we wanted to, for example, show all elements of a tree?

.. code-block:: haskell

    mapList :: (a -> b) -> List a -> List b
    mapTree :: (a -> b) -> Tree a -> Tree b
    gmap    :: (Mappable t) => (a -> b) -> t a -> t b

    class Functor where
        fmap :: (a -> b) -> t a -> t b

    instance Functor [] where
        fmap = mapList

    instance Functor Tree where
        fmap = mapList
