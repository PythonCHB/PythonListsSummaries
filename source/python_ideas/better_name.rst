##################
Add a Better Name?
##################

One issue that often comes up on ``python-ideas`` is to provide a "better" name for a module, function, etc in the standard library.  The truth is that "*naming is hard*", and the standard library was built up over the years and may not have ideal names for everything.  But while most would agree that many current names are not great, there are a lot of good reasons *not* to change names just to get a better name.

None the less, folks, often new to the language, sometimes find naming conventions confusing, and suggest that they be changed. The community is not completely opposed to these ideas, but there is a high bar to accept a change, just to provide a "better" name, even if everyone agrees that it is better.

Why not a new name?
===================

There are a lot of good reasons not to change names. In a thread about changing the name of ``json.loads`` -- a number of good explanations where provided:

https://mail.python.org/archives/list/python-ideas@python.org/thread/EJTIVQ2ZFSVHALTLRGFCOMOYGZYMKGQU/

Reading the thread is instructive, and not everyone agreed that``loads`` is a bad name, but here are a number of the key points provided that apply to any addition or changing of a name:

Steven D'Aprano Wrote:
----------------------

    I agree with you that this is a sub-optimal naming convention, and we
    would have been better if a different choice was made at the beginning.
    But unfortunately it is a widely used naming convention:

    * ``pickle``
    * ``marshall``
    * ``json``
    * ``yaml`` (I think)
    * and probably more

    and not just in Python. But I agree with the others that the pain and
    disruption from changing it is greater than the benefit. We all just
    have to memorize that "loads" means "load-string" and not the present
    tense of load.

Andrew Barnert wrote:
---------------------

    a previous poster wrote:
      > I'd be +1 on adding a better named alias for loading
      > strings to all of these libraries.

    Including the ones that aren’t in the stdlib? If so, how do we do that?

    If not, won’t it be even more confusing for people who learn json.load_string when they
    go to use PyYAML and there’s no yaml.load_string? Sure, they’ll complain to the PyYAML
    devs, and eventually, most of the most common libraries will catch up, but you’re talking
    about a few years of extra confusion before you get the long-term benefits.

    Also, all the existing code, tutorials, StackOverflow answers, etc. that uses loads
    will still be using loads, so most people are going to have to learn what it means
    anyway.

    People are still getting confused over find_all vs. findall and writing StackOverflow
    questions asking why some module is giving them a NameError when they try to use findall a
    decade later. And the same is true for some of the pep8-naming changes that came with 3.0;
    you can still find StackOverflow questions where the accepted answer on the canonical duplicate
    question says to use threading.currentThread even though current_thread was added, and
    currentThread turned into a compatibility alias, back in 2.6.

    So the Very Good Thing isn’t actually as good as you’d hope—people will still have to
    learn loads—and the downsides are bigger than they appear at first—it will be a years-long
    process to get the entire Python ecosystem consistent on a new naming convention.

    That doesn’t mean it definitely isn’t worth doing, but it does mean you need to argue
    that it’s compelling enough to be worth the tradeoff anyway, rather than ignoring the cost
    and arguing as if we had a time machine and could go back to whenever pickle was added and
    change history to avoid all the downsides.

Stephen J. Turnbull wrote:
--------------------------

    a previous poster wrote:
        > So I'm still +1. Just add the better named function as an option
        > and be done with it; other than adding it to the docs, leave
        > everything else as it is.

    I would find this very annoying, because I believe it would lead to
    widespread namespace pollution.  I use dir() a lot to remind myself of
    APIs, and aliases that would confuse me and slow reading (I'm old
    enough to have vision issues) would anger me.

    ...

    I'm basically -1 on any aliasing.  I think there's way too many that
    have been proposed.  I can't see a way limit them that doesn't read
    "screw you" to the proposals that are denied, and it would be a burden
    on me, it would cause confusion, and I don't think the benefits are
    likely to be great.

Chris Angelico wrote:
---------------------

    a previous poster wrote:
        > One of them can maybe be deprecated
        > ``json.load(<string here>)``
        > for file:
        > ``json.load(file=<file obj>)``

    Deprecation is not a solution. You're still going to have stuff out
    there using the old name (Stack Overflow posts, blogs, articles,
    books, etc etc etc etc), and people will have to learn the meaning
    of both. Additionally, churn like this - renaming things for no reason
    other than "the new name is somehow better" - tends to create the
    sense that the language is constantly in flux, and that today's code
    is going to be broken tomorrow. Imagine a Stack Overflow answer that
    has to be edited to say "For Python versions up to 3.8, use
    json.loads; for versions 3.9 and up, use json.load_string", and then
    if it uses a couple of other APIs that have been renamed, it needs to
    multiply that out exponentially... no thanks. I get enough of that
    from React.js.

    (-1) on renaming. (-1) on deprecating. (-1) on creating an unnecessary alias.

    a previous poster wrote:
        > My personal feeling is that adding a significantly better name, and
        > deprecating the old name (maybe never going past the state of
        > documenting that it is deprecated and suggesting using the better
        > name, one reason not to have an automatic removal of deprecated features) is
        > viable if the name improvement is big enough.

    Perhaps, IF the improvement is big enough. A marginal improvement is
    not worth all that hassle. And if you keep both names, you forever
    have people wondering what the difference is.

    As an example, Python's threading module has duplicate names e.g.
    "current_thread" and "currentThread". The preferred names are
    documented up to and including Python 2.7, but they still exist in the
    latest 3.x.

    ...

    a previous poster wrote:

        > One issue is that since Python allows the user to
        > monkey-patch library code, ANY expansion of name space is potentially
        > breaking, Python has almost no namespace reserved for the language
        > designers that users are allowed to intrude on (maybe dunders are
        > reserved enough, but we don't want to be using dunders as part of user
        > accessed API). Which means that even if we deprecate ``loads``, the standard
        > library should continue to use it and not the new load_string for awhile
        > until it is felt safe that few people are monkey patching in a
        > load_string member, and we can break the code.

    I don't think we need to worry about people poking arbitrary values
    into a stdlib module and breaking things. If someone's doing that and
    their code breaks in Python 3.10, that's on them.

Stephen J. Turnbull wrote:
--------------------------

    You don't have to like it.  I don't think anybody likes suboptimal
    naming, they just like it better than they like backward-incompatible
    renames or namespace pollution.  You can't really expect others to be
    happy with wholesale changes to the environment, even if they're fully
    backward compatible as the proposals to add aliases for ``json.loads`` and
    ``calendar.calendar`` are.  (As usual, you can say "I just want these`
    two," and we'll respond "OK, but the community as a whole will want a
    million of them, we all have our favorites, and if we accept yours,
    how do we deny theirs?")



.. rubric:: Authorship

Christopher H. Barker: ``PythonCHB@gmail.com``







