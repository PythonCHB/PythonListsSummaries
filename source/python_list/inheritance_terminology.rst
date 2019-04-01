##############################
Python Inheritance Terminology
##############################

`https://mail.python.org/pipermail/python-list/2018-January/729897.html <https://mail.python.org/pipermail/python-list/2018-January/729897.html>`_

I'm doing some writing for an upcoming course on OOP using Python.

I have been doing OOP programming for many years in many different languages, and I want make sure that I'm using the appropriate terminology in Python.  I'd like to know if there are "official" or even standard terms that are used to describe a class that is inherited from, and the class that is doing the inheriting.  From my reading (especially the PSF docs.python.org `http://docs.python.org/ <http://docs.python.org/>`_\ , it looks like the terms would be "base class" and "subclass".

However, in books about Python and other languages, I have also seen the terms:

base class & derived class
parent class & child class
superclass & subclass

So, are base class & subclass the proper terms?

responses
^^^^^^^^^

**1.**

Standard (“official”) terms are most likely to be had from the language
reference `http://docs.python.org/3/reference/ <http://docs.python.org/3/reference/>`_. I would recommend
the glossary `http://docs.python.org/3/glossary.html <http://docs.python.org/3/glossary.html>`_\ , but with the
caveat that many flaws have been found in recent years.

::

   > However, in books about Python and other languages, I have also seen the terms:
   >
   > base class & derived class
   > parent class & child class
   > superclass & subclass

The only term I take issue with there is “superclass”. In a
multiple-inheritance system, such as provided by Python, the superclass
is *not* necessarily the base class. See this article from 2011
`https://rhettinger.wordpress.com/2011/05/26/super-considered-super/ <https://rhettinger.wordpress.com/2011/05/26/super-considered-super/>`_.

::

   > So, are base class & subclass the proper terms?

In my opinion you will be correct to use those terms. Which is not to
say that other terms aren't also good.
