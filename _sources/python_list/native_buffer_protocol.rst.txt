NOTE: this page needs cleanign up and formatting in rst

Native object exposing buffer protocol
======================================

[https://mail.python.org/pipermail/python-list/2018-January/729896.html](https://mail.python.org/pipermail/python-list/2018-January/729896.html)

I'd like to create a native Python object that exposes the buffer
protocol.  Basically, something with a ```._data member``` which is a
bytearray that I can still read into, make directly into a numpy array, etc.

I can do it by inheriting the entire thing from bytearray directly, but
that gives me a whole lot of methods that are unsuitable for what I'm
doing with it, which is reading the contents of a binary file, allowing
them to be lightly malleable in memory, and then sending them on to a
device.

Not the end of the world (the file's less than 2KB), but it seems like
something that should be doable easily without having to throw around a
lot of extraneous copies.

Responses
---------

1.
..

Could you use memoryviews instead of making the copies?

2.
..

  > I'd like to create a native Python object that exposes the buffer
  > protocol.  Basically, something with a ._data member which is a
  bytearray that I can still readinto, make directly into a numpy array,
  etc.


The “etc.” seems pretty important, there. You want the behaviour of
‘bytearray’ without actually inheriting that behaviour from the
‘bytearray’ type.

So, it seems your options are:

* Enumerate all the things, specifically, that you do want your new type
  to do. Don't hide anything in “etc.”, so that you know exactly what
  behaviours need to be implemented. Implement all those behaviours,
  without benefit of inheriting from ``bytearray``.

* Inherit from ``bytearray``, but ameliorate the problems you want to
  avoid. This will require enumerating all those problems, so that you
  can know whether you have avoided them. Don't hide any of them in an
  “etc.”.


> Not the end of the world (the file's less than 2KB), but it seems like
> something that should be doable easily without having to throw around
> a lot of extraneous copies.


I look forward to your report from having tried it :-)

3. author
.........


  The “etc.” seems pretty important, there. You want the behaviour of
  ‘bytearray’ without actually inheriting that behaviour from the
  ‘bytearray’ type. >


Well, one specific behavior.  Ideally, what I want is, when calls like
RawIOBase.readinto and ndarray.frombuffer ask my class "Hey, do you have
any raw bytes that I can read and potentially write?" it can answer "Why
certainly, here they are."  Something like a:

.. code-block:: python

    class Thingy:
       def __memoryview__(self):
         return memoryview(self._data)

But it doesn't look as if there's a dunder for that.  There's ``__bytes__``, but that specifically requires you give back a bytes() object; even
returning a bytearray causes an error.


> So, it seems your options are:
>
> * Enumerate all the things, specifically, that you do want your new type
>    to do. Don't hide anything in “etc.”, so that you know exactly what
>    behaviours need to be implemented. Implement all those behaviours,
>    without benefit of inheriting from ‘bytearray’.
>
> * Inherit from ‘bytearray’, but ameliorate the problems you want to
>    avoid. This will require enumerating all those problems, so that you
>    can know whether you have avoided them. Don't hide any of them in an
>    “etc.”.


That's ultimately the way I'm going about it, but since this is only
going to get used in-house, I'm approaching it the Pythonic way.  The
problem is "Most methods of bytearray make no sense on a data structure
representing a fixed length waveform."  The solution is "Well, don't use
them."

4.
..

Curiosity having got the better of me I did some searching and found this "Implementing the buffer protocol" (http://cython.readthedocs.io/en/latest/src/userguide/buffer.html), any use to you?  Note that at the bottom of the link the final section "References" states "The buffer interface used here is set out in PEP 3118, Revising the buffer protocol.", i.e. it is the new protocol and not the old Python 2 one.
