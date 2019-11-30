#############
Stdlib Module
#############

This summarises what was described `here <https://mail.python.org/archives/list/python-ideas@python.org/thread/XSYEVRPJQUX7VBTPNIJMUFZQIZ7WLOQU/>`_ .

Abstract
========

Proposal: Adding a stdlib module to the standard library

Defining the stdlib
===================

Would the stdlib module be OS-independent or not? A better approach would be to go along the standard modules as defined in the `docs <https://docs.python.org/3/library/index.html>`_ . You get to know all modules within the stdlib, it even lists curses on windows (see benefit at 2. below) with info. But it does not let you access it if you want to use it when importing. Same for the third issue: 

The third issue is alternative Python implementations, discussed at concerns no.2.

Advantages of having a stdlib module
====================================

Below are some benefits of having such a module:

* 1 Knowing what module can be imported without installing

This can be useful for interactive learning. Like with the inbuilt builtins library, one get to know what builtins are there in Python, though it's not the lib's only use. In the spirit of the keyword module, it can also check if an import forms part of the std lib

* 2 Check for accidental name collisions

It might come in handy to check for python platform collisions. An intersection between their modules will do the job

* 3 Namespaced imports

.. code-block:: python

    import math

could be used as

.. code-block:: python

    stdlib.math

to better distinguish.

Concerns
========

* 1. Backward compatibility

Python 4.0 codes might contain codes incompatible with previous versions

* 2. Different standard modules

If a Python implementation adds in a module, would codes implementing

stdlib.fancy_lib

on a diffrent platform not get broken? That is essentially the same as executing curses on windows or RPi on Raspberry Pi * Raspian. Just like RPi codes would not run on a windows, similarly, stdlib.fancy_lib would not run on the standard C implementation.


Ideas and Implementations
=========================

* Andrew Barnert implemented a `version of stdlib <https://github.com/abarnert/stdlib>`_ where you can import without having to imply where the module is meaning instead of from x.y import z, you do:
from stdlib import z

* Each implementation could hardcode the names of std modules which could be accessed by stdlib for listing and imports

* Use what help('modules') uses i.e pkgutil.walk_packages minus 3rd party modules

* This `stdlib implemenation <https://github.com/jackmaney/python-stdlib-list>`_ from Jack Maney essentially hardcodes the libs like `here <https://github.com/jackmaney/python-stdlib-list/blob/master/stdlib_list/lists/3.7.txt>`_.

* The real implementation is up to ... be implemented.

Draft containing inputs from:

* Andrew Barnert

* Chris Angelico

* Christopher Barker

* Steve Barnes

* Steven D'Aprano

Abdur-Rahmaan Janhangeer: ``arj.python@gmail.com``