#####################
Join as a list method
#####################

It is often brought up that the method for joining a sequence of strings should be a method of the list object (or all sequences), rather than a method on strings. That is:

.. code-block:: python

    ["Some", "words" "to" "put" "together"].join(" ")

rather than:

.. code-block:: python

    " ".join(["Some", "words" "to" "put" "together"])

This was recently brought up in this thread:

https://mail.python.org/pipermail/python-ideas/2019-March/056064.html

Conclusion
==========

It is very unlikely that ``.join`` will ever become a list method. It was (and is) a deliberate design decision.

Justification
=============

``join`` was originally a function in the ``string`` module. It Python 2.0, the ``unicode`` type was added, most of the functions in the string module became methods on both the ``str`` and ``unicode`` types, providing a single interface for string processing.
While in Python3, the string types have been unified, it still makes sense with Python's design that all string related functionality be found in the ``str`` object.

1. String joining is inherently a string operation.
   Lists can be used to hold any data type. Having a method on list that only makes sense for a small fraction of the list use-cases makes little sense.
   There are no other list methods that can only be used with certain types.

2. A ``join`` method could call ``str()`` on all items, but while Python will return *something* for all data types, not everything would make sense.
   And even for data types that do make sense, users are likely to want to control how the items are "stringified" -- with string formatting, etc.

3. ``str.join()`` supports not just lists (or other sequences), but any iterable.
   This has become particularly important in Python3, which has shifted focus toward working with iterables, rather than requiring the creation of sequences simply to iterate through them.
   So why not make it an iterable method? because iterables are not a type, but rather anything that conforms to the iteration protocol
   (https://treyhunner.com/2016/12/python-iterator-protocol-how-for-loops-work/).
   As there is no way to add ``.join()`` to iterators. Moving ``.join()`` to the sequence ABC would result in losing important functionality, and require that an iterable be turned into a sequence before joining it -- a potentially significant performance hit, and contrary to python 3's design.
   Having ``.join()`` as both a list and string method would create confusion for little gain.









