# PythonListsSummaries
A set of pages summarizing discussion on major Python mailing lists.

Published here:

https://pythonchb.github.io/PythonListsSummaries/

## Project Motivation

The Python mailing lists are a real treasure trove with gems scattered here and there. Unfortunately, this mine of info lays hidden, buried as years pass by.

- Referring someone to a past question is a pain. You have to dig it out.
- The reading format is a bit dated (the plain archive).
- And ... good luck following the 112 messages to get the gist of the issue.

This project aims at providing rich articles summarizing threads, a reference for the community! But, we need **your** help!

For how it started, please refer [here](https://mail.python.org/pipermail/python-ideas/2019-March/056071.html) and [here](https://mail.python.org/pipermail/python-ideas/2019-March/056123.html).

## Contributing

These pages are intended to be community supported.  If you have a new page
you'd like to contribute, or have suggestions for improving existing pages,
please post and issue, or better yet, a PR on this repo.

### How These Docs are Built

These pages are built with the [Sphinx Documentation System](http://www.sphinx-doc.org/en/master/)

Sphinx is widely used in the Python Community, so a good choice for these pages, and a valuable tool to learn for any Pythonista.

Sphinx pages are written using [Restructured Text](http://docutils.sourceforge.net/rst.html), a text-based markup format similar to Markdown.
Refer to the Sphinx docs for details, but in short:

#### Building the docs:

You need to have sphinx installed, which you can do with:

```
pip install -r requirements.txt
```

You can then build the docs with:

```
make html
```

It's a good idea to build them on your local machine to make sure that you are happy with the formatting before making a PR to the repo.
The published version is built automatically by Travis CI when new content is pushed to master.
See the `.travis.yml` file for how that's done.

### Adding a new page

For working with git, gitHub, and Pull Requests, see other sources -- google is your friend.

When adding a new page, you need to create a file in the appropriate directory, called: `something_meaningfull.rst` -- see the existing pages for how that should be structured.
Then you can add a line to the `index.rst` page in that dir so that it will be listed in teh table of contents.

You may want to add your name as the author at the bottom of the page, and/or add your name to the list if you make a major contribution to an existing page.

