##########################
Where did csv.parser() go?
##########################

`<https://mail.python.org/pipermail/python-list/2018-January/729851.html>`_

Question
^^^^^^^^

I need record the starting offsets of csv rows in a database for fast seeking later.
Unfortunately, using any csv.reader() (or DictReader) tries to cache, which means:

.. code-block:: python

   example_Data = "'data
   0123456789ABCDE
   1123456789ABCDE
   2123456789ABCDE
   3123456789ABCDE
   ...


   for line in reader:
       offsets[row] = f.tell()

is not possible. With my example data , offsets are reported as ``[0, 260, 260, 260...]`` they should be ``[0x00, 0x00,0x15, 0x25, ...]`` (sample data is 16 byte rows after a 5 byte header (just for now))

I saw in one of PEP-305's references a mention of csv.parser() which won't return a row until parsing is complete. This is ideal since some lines will have quoted text containing commas and new lines.  I don't want to re-write the parser, since later usage will use csvDictReader, so we need to identically parse rows. How can I do that with the Python 2.7 csv module?

Or how can I accomplish this task through other means?

Response
^^^^^^^^

It's not the reader that performs the caching it's iteration over the file:

.. code-block:: bash

   $ python -c 'f = open("tmp.csv")
   > for line in f: print f.tell()
   > '
   73
   73
   73
   73
   73
   73

You can work around that by using the file's readline() method:

.. code-block:: bash

   $ python -c 'f = open("tmp.csv")
   for line in iter(f.readline, ""): print f.tell()
   '
   5
   21
   37
   53
   69
   73

Combined with csv.reader():

.. code-block:: bash

   $ python -c 'import csv; f = open("tmp.csv")
   for row in csv.reader(iter(f.readline, "")): print f.tell(), row
   '
   5 ['data']
   21 ['0123456789ABCDE']
   37 ['1123456789ABCDE']
   53 ['2123456789ABCDE']
   69 ['3123456789ABCDE']
   73 ['...']
