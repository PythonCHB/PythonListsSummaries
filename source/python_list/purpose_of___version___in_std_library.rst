###############################################################
What is the purpose of ``__version__`` in the standard library?
###############################################################

`<https://mail.python.org/pipermail/python-list/2019-March/739765.html>`_

A question. I help maintain a Python stack for users in my division here 
at NASA and one user asked about updating the re module to 2.4. I 
believe because he read the docs:

https://docs.python.org/2.7/library/re.html

where you see lines like "New in version 2.4" and he also did:

::

    $ python2 -c 'import re; print (re.__version__)'
    2.2.1

And, well, one can think "oh, a newer version is needed". I searched on 
conda, etc. and can't find it and finally realized that 2.4 meant Python 
2.4, not re 2.4. (The 3.7 docs have lines like "Changed in version 3.7".)

My question to the pros here is what purpose do the ``__version__`` /version 
variables serve in the Python Standard Library?  I can understand in 
external packages, but once in the Standard Library...?

For example, in re.py, that line was last changed 18 years ago according 
to git blame. In tarfile.py, the version string was last changed 12 
years ago. But in both, the modules were edited in 2018 so they haven't 
been static for a decade.

Are those strings there just for historic purposes?

Note:

::

    Effectively a re.__version__ in python 3.7 still tells 2.2.1

answer
^^^^^^

The point of the "Changed in version ..." or "New in version ..." bits
in the documentation is to alert readers who maintain software which
needs to remain backward compatible with older versions of Python. If
you maintain a package which you support for Python 3.4, 3.5, 3.6, and
3.7, you'll probably shy away from bits which weren't around for 3.4,
and be careful about APIs which have changed since 3.4.

If a module that started life outside the standard library is included
in the standard library under the same name, it is taken aboard hook,
line and sinker. If it has a ``__version__``, then that should probably
stay: code that used the module before it entered the standard library
might rely on it.

As you quite rightly point out, these lines are then completely
pointless, so there's no reason to ever change them.