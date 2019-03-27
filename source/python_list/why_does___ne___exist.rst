##########################
Why does ``__ne__`` exist?
##########################

`<https://mail.python.org/pipermail/python-list/2018-January/729918.html>`_

When you create a Python class, you can create dunder methods to
define how your objects respond to the standard operators. With
comparison operators, Python will happily switch the operands around
to find a method to call:

.. code-block:: python

   >>> class Spam():
   ...     def __lt__(self, other):
   ...         print("%s is less than %s" % (self, other))
   ...         return True
   ...
   >>> Spam() < 2
   <__main__.Spam object at 0x7fb7557b1fd0> is less than 2
   True
   >>> 3 > Spam()
   <__main__.Spam object at 0x7fb7557b1fd0> is less than 3
   True
   >>> 4 > Spam() < 5
   <__main__.Spam object at 0x7fb7557b1fd0> is less than 4
   <__main__.Spam object at 0x7fb7557b1fd0> is less than 5
   True

But Python will not automatically assume the converse:

.. code-block:: python

   >>> Spam() >= 6
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   TypeError: '>=' not supported between instances of 'Spam' and 'int'
   >>> Spam() > 7
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   TypeError: '>' not supported between instances of 'Spam' and 'int'

This is good. This is correct. For inequalities, you can't assume that
``>=`` is the exact opposite of ``<`` or the combination of ``<`` and ``==`` (for
example, sets don't behave like numbers, so ``x <= y`` is very different
from ``x < y or x == y``\ ). But the one that confuses me is ``!=`` or ``__ne__``.
If you don't create it, you get default behaviour:

.. code-block:: python

   >>> class Ham:
   ...     def __eq__(self, other):
   ...         print("%s equals %s" % (self, other))
   ...         return True
   ...
   >>> Ham() == 1
   <__main__.Ham object at 0x7fb7557c0278> equals 1
   True
   >>> 2 == Ham()
   <__main__.Ham object at 0x7fb7557c0278> equals 2
   True
   >>> Ham() != 3
   <__main__.Ham object at 0x7fb7557c0278> equals 3
   False
   >>> 4 != Ham()
   <__main__.Ham object at 0x7fb7557c0278> equals 4
   False
   >>> x = Ham()
   >>> x == x
   <__main__.Ham object at 0x7fb7557b80f0> equals <__main__.Ham object at
   0x7fb7557b80f0>
   True
   >>> x != x
   <__main__.Ham object at 0x7fb7557b80f0> equals <__main__.Ham object at
   0x7fb7557b80f0>
   False

Under what circumstances would you want ``x != y`` to be different from
``not (x == y)`` ? How would this make for sane behaviour? Even when
other things go weird with equality checks, that basic parallel is
always maintained:

.. code-block:: python

   >>> z = float("nan")
   >>> z == z
   False
   >>> z != z
   True

Python gives us a ``not in`` operator that uses ``__contains__`` and then
negates the result. There is no way for ``x not in y`` to be anything
different from ``not (x in y)``\ , as evidenced by the peephole optimizer:

.. code-block:: python

   >>> dis.dis("x not in y")
     1           0 LOAD_NAME                0 (x)
                 2 LOAD_NAME                1 (y)
                 4 COMPARE_OP               7 (not in)
                 6 RETURN_VALUE
   >>> dis.dis("not (x in y)")
     1           0 LOAD_NAME                0 (x)
                 2 LOAD_NAME                1 (y)
                 4 COMPARE_OP               7 (not in)
                 6 RETURN_VALUE

So why isn't ``!=`` done the same way? Is it historical?

responses
^^^^^^^^^

**1.**

::

   > Under what circumstances would you want "x != y" to be different from
   > "not (x == y)" ?

In numpy, ``__eq__`` and ``__ne__`` do not, in general, return bools.

.. code-block:: python

   Python 3.6.3 (default, Oct  3 2017, 21:45:48)
   [GCC 7.2.0] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>> import numpy as np
   >>> a = np.array([1,2,3,4])
   >>> b = np.array([0,2,0,4])
   >>> a == b
   array([False,  True, False,  True], dtype=bool)
   >>> a != b
   array([ True, False,  True, False], dtype=bool)
   >>> not (a == b)
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   ValueError: The truth value of an array with more than one element is
   ambiguous. Use a.any() or a.all()
   >>> ~(a == b)
   array([ True, False,  True, False], dtype=bool)
   >>>

I couldn't tell you why this was originally allowed, but it does turn
out to be strangely useful. (As far as the numpy API is concerned, it
would be even nicer if 'not' could be overridden, IMHO)

**2. author**

Thanks, that's the kind of example I was looking for. Though numpy
doesn't drive the core language development much, so the obvious next
question is: was this why ``__ne__`` was implemented, or was there some
other reason? This example shows how it can be useful, but not why it
exists.

.. code-block:: python

   >>>> not (a == b)
   > Traceback (most recent call last):
   >   File "<stdin>", line 1, in <module>
   > ValueError: The truth value of an array with more than one element is
   > ambiguous. Use a.any() or a.all()

Which means that this construct is still never going to come up in good code.

::

   >>>> ~(a == b)
   > array([ True, False,  True, False], dtype=bool)
   >>>>
   >
   > I couldn't tell you why this was originally allowed, but it does turn
   > out to be strangely useful. (As far as the numpy API is concerned, it
   > would be even nicer if 'not' could be overridden, IMHO)

I'm glad ``not`` can't be overridden; it'd be too hard to reason about a
piece of code if even the basic boolean operations could change. If
you want overridables, you have ``&|~`` for the bitwise operators (which
is how numpy does things).

Has numpy ever asked for a ``not in`` dunder method (\ ``__not_contained__``
or something)? It's a strange anomaly that "not (x in y)" can be
perfectly safely optimized to ``x not in y``\ , yet basic equality has to
have separate handling. The default handling does mean that you can
mostly ignore ``__ne__`` and expect things to work, but if you subclass
something that has both, it'll break:

.. code-block:: python

   class Foo:
       def __eq__(self, other):
           print("Foo: %s == %s" % (self, other))
           return True
       def __ne__(self, other):
           print("Foo: %s != %s" % (self, other))
           return False

   class Bar(Foo):
       def __eq__(self, other):
           print("Bar: %s == %s" % (self, other))
           return False

   >>> Bar() == 1
   Bar: <__main__.Bar object at 0x7f40ebf3a128> == 1
   False
   >>> Bar() != 1
   Foo: <__main__.Bar object at 0x7f40ebf3a128> != 1
   False

Obviously this trivial example looks stupid, but imagine if the
equality check in the subclass USUALLY gives the same result as the
superclass, but differs in rare situations. Maybe you create a "bag"
class that functions a lot like collections.Counter but ignores zeroes
when comparing:

.. code-block:: python

   >>> class Bag(collections.Counter):
   ...     def __eq__(self, other):
   ...         # Disregard any zero entries when comparing to another Bag
   ...         return {k:c for k,c in self.items() if c} == {k:c for k,c
   in other.items() if c}
   ...
   >>> b1 = Bag("aaabccdd")
   >>> b2 = Bag("aaabccddq")
   >>> b2["q"] -= 1
   >>> b1 == b2
   True
   >>> b1 != b2
   True
   >>> dict(b1) == dict(b2)
   False
   >>> dict(b1) != dict(b2)
   True

The behaviour of ``__eq__`` is normal and sane. But since there's no
``__ne__``\ , the converse comparison falls back on ``dict.__ne__``\ , not on
``object.__ne__``.

**3.**

::

   >> In numpy, __eq__ and ```__ne__``` do not, in general, return bools.
   >>
   >>>>> a = np.array([1,2,3,4])
   >>>>> b = np.array([0,2,0,4])
   >>>>> a == b
   >> array([False,  True, False,  True], dtype=bool)
   >>>>> a != b
   >> array([ True, False,  True, False], dtype=bool)
   >
   > Thanks, that's the kind of example I was looking for. Though numpy
   > doesn't drive the core language development much, so the obvious next
   > question is: was this why ```__ne__``` was implemented, or was there some
   > other reason? This example shows how it can be useful, but not why it
   > exists.

Actually, I think it is why it exists.  If I recall correctly, the addition of the six comparative operators* was added
at the behest of the scientific/numerical community.

**4.**

The six rich comparison methods were added to 2.1 as a result of PEP 207, which confirms that you're correct, they were added at the request of the numpyites.

**5.**

Interesting sentence from that PEP:

::

   "3. The == and != operators are not assumed to be each other's
   complement (e.g. IEEE 754 floating point numbers do not satisfy this)."

Does anybody here know how IEE 754 floating point numbers need ``__ne__``\ ?

**6.**

That's very interesting. I'd also like an answer to this. I can't wrap
my head around why it would be true. I've just spent 15 minutes playing
with the interpreter (i.e. checking operations on 0, -0, 7,
``float('nan')``\ , ``float('inf')``\ , etc.) and then also reading a bit about IEEE
754 online and I can't find any combination of examples where ``==`` and ``!=``
are not each others' complement.

**7.**

I don't see a case in IEEE where ``(x == y) != !(x != y)``.
There *is* a case where ``(x != x)`` is true ``(when x is NaN)``\ , but for such an
x, ``(x == x)`` will be false.

I am hard pressed to think of a case where ``__ne__`` is actually useful.

That said, while it is true you only need one of (\ ``__eq__``\ , ``__ne__``\ ), you
could make the same claim about (\ ``__lt__``\ , ``__ge__``\ ) and (\ ``__le__``\ , ``__gt__``\ ).
That is, in principle you could get by with only (\ ``__eq__``\ , ``__le__``\ , and
``__ge__``\ ) or, if you prefer, (\ ``__ne__``\ , ``__lt__``\ , ``__gt__``\ ), or any other
combination you prefer.

Or you could go where C++ is doing and say that *if* one specifies a single
``__cmp__`` method, it should return one of LT, EQ, GT, and this will
automatically give rise to all the comparison operators.

"Trade-offs... trafe-offs as far as the eye can see" ;-)

**8.**

Assuming you're talking about a case specifically for IEEE 754, I'm
starting to agree. In general, however, it certainly is useful for some
numpy objects (as mentioned elsewhere in this thread).

::

   > That said, while it is true you only need one of (__eq__, __ne__), you
   > could make the same claim about (__lt__, __ge__) and (__le__, __gt__).
   > That is, in principle you could get by with only (__eq__, __le__, and
   > __ge__) or, if you prefer, (__ne__, __lt__, __gt__), or any other
   > combination you prefer.

This isn't true for IEEE 754. For example:

.. code-block:: python

   >>> float('nan') < 0
   False
   >>> float('nan') > 0
   False
   >>> float('nan') == 0
   False

Also there are many cases where you don't have a < b OR a >= b. For
example, subsets don't follow this.

::

   > "Trade-offs... trafe-offs as far as the eye can see" ;-)

Yes few things in life are free. :)

**9.**

Ugh, right, for NaN you can have ``(x < y) != (x >= y)`` - both would be false
if one of x and y is a NaN.

But ``__ne__`` is still useless ;-)

**10.**

::

   > I don't see a case in IEEE where (x == y) != !(x != y).
   > There _is_ a case where (x != x) is true (when x is NaN), but for such an
   > x, (x == x) will be false.
   >
   > I am hard pressed to think of a case where ```__ne__``` is actually useful.

See my earlier email and/or PEP 207. (tl;dr: non-bool return values)

::

   > That said, while it is true you only need one of (__eq__, __ne__), you
   > could make the same claim about (__lt__, __ge__) and (__le__, __gt__).
   > That is, in principle you could get by with only (__eq__, __le__, and
   > __ge__) or, if you prefer, (__ne__, __lt__, __gt__), or any other
   > combination you prefer.

PEP 207: "The above mechanism is such that classes can get away with not
implementing either ``__lt__`` and ``__le__`` or ``__gt__`` and ``__ge__``."

::

   > Or you could go where C++ is doing and say that _if_ one specifies a single
   > __cmp__ method, it should return one of LT, EQ, GT, and this will
   > automatically give rise to all the comparison operators.

This used to be the case. (from version 2.1 to version 2.7, AFAICT)

**11.**

::

   > Actually, I think it is why it exists.  If I recall correctly, the
   > addition of the six comparative operators* was added at the behest of
   > the scientific/numerical community.

Which personnaly, I think was a mistake. I can understand it is useful
for the scientific/numerical community to compare vectors with each
other and get a vector of booleans. However how useful is it doing this
with the normal boolean operators, instead of calling a function?

And if doing it with an operator was so important, I think it would have
been better to introduce boxed operators, like ``[+]``\ , ``[<]`` ... where the
default behaviour would be an elementary wise application of the
non-boxed operator.

**12.**

::

   >> In numpy, __eq__ and ```__ne__``` do not, in general, return bools.
   >>
   >>>>> a = np.array([1,2,3,4])
   >>>>> b = np.array([0,2,0,4])
   >>>>> a == b
   >> array([False,  True, False,  True], dtype=bool)
   >>>>> a != b
   >> array([ True, False,  True, False], dtype=bool)
   >
   > Thanks, that's the kind of example I was looking for. Though numpy
   > doesn't drive the core language development much, so the obvious next
   > question is: was this why ```__ne__``` was implemented, or was there some
   > other reason? This example shows how it can be useful, but not why it
   > exists.

AFAIK this was the main reason. This can be also used for creating queries.

NumPy inspired 4 or 5 core features which are rarely used outside of
NumPy. They include the possibility of comparison operators to return
non-booleans

**13.**

::

   > Under what circumstances would you want "x != y" to be different from
   > "not (x == y)" ? How would this make for sane behaviour?

Presumably so that any behaviour any be programmed when overriding these
operators.

Maybe someone wants to do weird stuff with ``==`` that doesn't yield a true
or false result, so that you can't just reverse it for ``!=``.

For example (perhaps this is similar to what was suggested in another post):

::

     (10,20,30) == (10,20,40)   yields  (1,1,0)
     (10,20,30) != (10,20,40)   yields  (0,0,1)

Although here, you would probably define 'not' so that ``not (1,1,0)``
does actually yield ``(0,0,1)``.

So clearly I need a weirder example.

  Even when

::

   > other things go weird with equality checks, that basic parallel is
   > always maintained:
   >
   >>>> z = float("nan")
   >>>> z == z
   > False
   >>>> z != z
   > True
   >
   > Python gives us a "not in" operator that uses __contains__ and then
   > negates the result. There is no way for "x not in y" to be anything
   > different from "not (x in y)", as evidenced by the peephole optimizer:
   >
   >>>> dis.dis("x not in y")
   >    1           0 LOAD_NAME                0 (x)
   >                2 LOAD_NAME                1 (y)
   >                4 COMPARE_OP               7 (not in)
   >                6 RETURN_VALUE
   >>>> dis.dis("not (x in y)")
   >    1           0 LOAD_NAME                0 (x)
   >                2 LOAD_NAME                1 (y)
   >                4 COMPARE_OP               7 (not in)

I get '4 COMPARE OP    6 (in)' here. So they are distinct ops. 'not in'
doesn't just call 'in', then apply 'not'. Not here anyway.

**14. author**

With tuples, I absolutely agree with Python's current behaviour: the
tuples you give are simply not equal. A tuple doesn't represent a
vector; it represents a specific collection of values, like the
coordinates of a point in 2D or 3D space. If you look at the two
points (1,5) and (3,5), they aren't "half equal". They're different
points, at different locations. They happen to have the same
elevation, but that's just a point of curiosity.

::

   >  Even when
   >>
   >> other things go weird with equality checks, that basic parallel is
   >> always maintained:
   >>
   >>>>> z = float("nan")
   >>>>> z == z
   >>
   >> False
   >>>>>
   >>>>> z != z
   >>
   >> True
   >>
   >> Python gives us a "not in" operator that uses __contains__ and then
   >> negates the result. There is no way for "x not in y" to be anything
   >> different from "not (x in y)", as evidenced by the peephole optimizer:
   >>
   >>>>> dis.dis("x not in y")
   >>
   >>    1           0 LOAD_NAME                0 (x)
   >>                2 LOAD_NAME                1 (y)
   >>                4 COMPARE_OP               7 (not in)
   >>                6 RETURN_VALUE
   >>>>>
   >>>>> dis.dis("not (x in y)")
   >>
   >>    1           0 LOAD_NAME                0 (x)
   >>                2 LOAD_NAME                1 (y)
   >>                4 COMPARE_OP               7 (not in)
   >
   >
   > I get '4 COMPARE OP    6 (in)' here. So they are distinct ops. 'not in'
   > doesn't just call 'in', then apply 'not'. Not here anyway.
   >

The fact that your Python doesn't optimize it is actually beside the
point; if *any* Python interpreter can optimize this down, it must be
semantically identical. I did this on CPython 3.7, fwiw, but it
doesn't really matter.

**15.**

::

   >> Maybe someone wants to do weird stuff with == that doesn't yield a true or
   >> false result, so that you can't just reverse it for !=.
   >>
   >> For example (perhaps this is similar to what was suggested in another post):
   >>
   >>   (10,20,30) == (10,20,40)   yields  (1,1,0)
   >>   (10,20,30) != (10,20,40)   yields  (0,0,1)

   > With tuples, I absolutely agree with Python's current behaviour: the
   > tuples you give are simply not equal. A tuple doesn't represent a
   > vector; it represents a specific collection of values, like the
   > coordinates of a point in 2D or 3D space. If you look at the two
   > points (1,5) and (3,5), they aren't "half equal". They're different
   > points, at different locations. They happen to have the same
   > elevation, but that's just a point of curiosity.

My ``(10,20,30)`` were meant to represent some user-defined type, not an
ordinary tuple. And someone might intend that ``==`` operates on two
instances of that type as thought they were vectors. Or any other kind
of behaviour as I said.

But not necessarily some logical inverse of ``!=``.

::

   >>>>>> dis.dis("not (x in y)")
   >>>
   >>>     1           0 LOAD_NAME                0 (x)
   >>>                 2 LOAD_NAME                1 (y)
   >>>                 4 COMPARE_OP               7 (not in)
   >>
   >>
   >> I get '4 COMPARE OP    6 (in)' here. So they are distinct ops. 'not in'
   >> doesn't just call 'in', then apply 'not'. Not here anyway.
   >>
   >
   > The fact that your Python doesn't optimize it is actually beside the
   > point; if _any_ Python interpreter can optimize this down, it must be
   > semantically identical.

Actually I didn't see the 'not' on the outside of the brackets. I
thought the two expressions were 'not in' and 'in' and that you might
have transcribed the '7 (not in)' part wrongly.

But this reduction isn't necessarily an optimisation. It might just be a
syntactical transformation from 'not (a in b)' to '(a not in b)'

The disassembly for 'in' and 'not in' suggests that these are two
independent operators, which could indeed have behaviours that are not
complements of each other.

On the other hand, when you /did/ want to evaluate 'in' followed by
'not', then you want:

::

       not (a in b)            # compare using 'not in'

to do the same thing as:

::

       temp = a in b           # compare using 'in'
       not temp                # apply unary not

Then there might not be the freedom to have in/not in have independent
behaviours.

**16. author**

::

   >>>>>>> dis.dis("not (x in y)")
   >>>>
   >>>>
   >>>>     1           0 LOAD_NAME                0 (x)
   >>>>                 2 LOAD_NAME                1 (y)
   >>>>                 4 COMPARE_OP               7 (not in)
   >>>
   >>>
   >>>
   >>> I get '4 COMPARE OP    6 (in)' here. So they are distinct ops. 'not in'
   >>> doesn't just call 'in', then apply 'not'. Not here anyway.
   >>>
   >>
   >> The fact that your Python doesn't optimize it is actually beside the
   >> point; if _any_ Python interpreter can optimize this down, it must be
   >> semantically identical.
   >
   >
   > Actually I didn't see the 'not' on the outside of the brackets. I thought
   > the two expressions were 'not in' and 'in' and that you might have
   > transcribed the '7 (not in)' part wrongly.

That's why I don't transcribe - I copy and paste. It's way WAY safer that way.

::

   > But this reduction isn't necessarily an optimisation. It might just be a
   > syntactical transformation from 'not (a in b)' to '(a not in b)'
   >
   > The disassembly for 'in' and 'not in' suggests that these are two
   > independent operators, which could indeed have behaviours that are not
   > complements of each other.

Uhm, if the peephole optimizer does a syntactical transformation, it
MUST retain the semantics. The disassembly for "in" and "not in" shows
that they are independent, but the disassembly for "not (x in y)"
proves that they are semantically linked.

::

   > On the other hand, when you /did/ want to evaluate 'in' followed by 'not',
   > then you want:
   >
   >    not (a in b)            # compare using 'not in'
   >
   > to do the same thing as:
   >
   >    temp = a in b           # compare using 'in'
   >    not temp                # apply unary not
   >
   > Then there might not be the freedom to have in/not in have independent
   > behaviours.

And the whole point of my post is that there is no such freedom - that
"not in" MUST always give the exact converse of "in". (And if
``__contains__`` returns something other than a boolean, it is coerced
before the operator returns it.) Yet equality is not like that. Hence
my post.

**17.**

::

   > So why isn't != done the same way? Is it historical?

I'd say this is certainly historical, remembering that in Python 2 you used to be able to compare all sorts of things, whereas in Python 3 you'll get:-

.. code-block:: python

   >>> 1 < "a"
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   TypeError: '<' not supported between instances of 'int' and 'str'
   >>>

This seems to be confirmed by the following.  From the third paragraph at `https://docs.python.org/2/reference/datamodel.html#object.__ne__ <https://docs.python.org/2/reference/datamodel.html#object.__ne__>`_

::

   "There are no implied relationships among the comparison operators. The truth of x==y does not imply that x!=y is false. Accordingly, when defining __eq__(), one should also define __ne__() so that the operators will behave as expected...".  Compare that with the Python 3 equivalent "By default, __ne__() delegates to __eq__() and inverts the result unless it is NotImplemented. There are no other implied relationships among the comparison operators, for example, the truth of ```(x<y or x==y)``` does not imply ```x<=y```..."

**18. author**

Ah, I forgot to check the Py2 docs. So, yeah, sounds like it's
basically historical. I'm still not sure why it was done in the first
place, but it looks like it's the sort of thing that wouldn't be done
now.

**19.**

I'm not understanding why you speculate that it wouldn't be done today.

We've established that it is useful to allow data types to define their
own meaning of “equal” and “not equal”, like many other operations. Is
that not good enough reason to allow it still?

**20. author**

The fact that container types can define "contains" but can't define
"doesn't contain", and that (as of Py3) there's proper default
handling, suggests that it's not as big a priority now.

Let's put it this way. Suppose that **eq** existed and ``__ne__`` didn't,
just like with **contains**. Go ahead: sell the notion of **ne**.
Pitch it, show why we absolutely need to allow this. Make sure you
mention the potential confusion when subclassing. Be sure to show why
it's okay for "not in" to force to boolean but "==" should allow any
return value.

**21.**

::

   > The fact that container types can define "contains" but can't define
   > "doesn't contain", and that (as of Py3) there's proper default
   > handling, suggests that it's not as big a priority now.

That is an inconsistency, I agree.

::

   > Let's put it this way. Suppose that __eq__ existed and ```__ne__``` didn't,
   > just like with __contains__. Go ahead: sell the notion of __ne__.
   > Pitch it, show why we absolutely need to allow this.

I think “reject unless absolutely needed” is an unreasonably high bar,
which would disqualify most Python language features. So I don't know
why you expect this to be so especially strongly argued.

::

   > Make sure you mention the potential confusion when subclassing.

For example, that would also be a problem for multiple inheritance. Not
“absolutely needed”, and high risk of confusion when subclassing. Do you
think that multiple inheritance would thereby also not be allowed today?

If you consider that a different case, why?

**22. author**

There's a LOT that you can do usefully with MI that you can't do
without it. Having spent a few years (many years ago) working with
Java, I appreciate the ability to inherit from more than one class.
Does it have to be done the way Python currently does it? No. But one
way or another, it's a massively useful feature.

(You're right that "absolutely needed" is too high a bar, but
hyperbole aside, I do think that MI hits a higher mark than **ne**
does.)

**23.**

I'm not the one making pronouncements on what would or would not be
allowed, in a counterfactual universe where things had been different
for so many years. So, because I don't need to speculate about that, I
won't :-)

**24.**

Considering we just recently added a matrix-multiplication operator, yes.

**25. author**

That has approximately zero consequences on class authors. If you were
unaware of ``__matmul__``, it wouldn't have the chance to randomly break
your ``__mul__`` semantics. And even with that excellent backward
compatibility, it STILL took many years to get accepted.

**26.**

::

   > Let's put it this way. Suppose that __eq__ existed and __ne__ didn't,
   > just like with __contains__. Go ahead: sell the notion of __ne__.
   > Pitch it, show why we absolutely need to allow this. Make sure you
   > mention the potential confusion when subclassing. Be sure to show why
   > it's okay for "not in" to force to boolean but "==" should allow any
   > return value.

``__ne__`` and ``__eq__`` are important for building mask arrays in NumPy,
which allow complex indexing operations.  A lot of NumPy's design was
inspired by MATLAB, so being able to index the same way as in MATLAB
is a pretty killer feature.

Indexing an array using mask arrays like this is idiomatic:

::

   some_arr = np.array([-1, 0, 1, 2, 3, 4, 5, 2, -1, 3, -1, 6, 7, 3])
   valid = some_arr[some_arr != -1]

Anybody with familiarity with NumPy appreciates that this is possible.

I imagine that ORMs like Django or SqlAlchemy also override ``__ne__`` to
provide nice APIs.

Finally (and perhaps least imporant), there is a performance hit if
only allowing ``__eq__`` and then taking its inverse.

**27.**

::

   > some_arr = np.array([-1, 0, 1, 2, 3, 4, 5, 2, -1, 3, -1, 6, 7, 3])
   > valid = some_arr[some_arr != -1]

I guess it was rather useful because list comprehension wasn't included
in the language at that moment, but I don't find it that much more useful
than:

::

      valid = somearr[[number != -1 for number in somearr]]

**28. author**

::

   > Anybody with familiarity with NumPy appreciates that this is possible.

I've used it, and I'm familiar with it, and I'm still not sure that I
appreciate it. But if it's there because of MATLAB, well, I guess
that's what people want?
