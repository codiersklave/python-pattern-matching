Python Pattern Matching
=======================

Python, I love you. But I'd like you to change. It's not you, it's me. Really.
See, you don't have pattern matching. But, that's not the root of it. Macros are
the root of it. You don't have macros but that's OK. Right now, I want pattern
matching. I know you offer me ``if``/``elif``/``else`` statements but I need
more. I'm going to abuse your ``with`` statements. Guido, et al, I hope you can
forgive me. This will only hurt a little.

Presenting: pattern matching in Python.

A lot of people have tried to make this work before. Somehow it didn't take. I
should probably call this yet-another-python-pattern-matching-module but
"yappmm" doesn't roll off the tongue. Other people have tried overloading
operators and changing codecs. This module started as a codec hack but those are
hard because they need an ecosystem of emacs-modes, vim-modes and the like to
really be convenient.

Python Pattern Matching takes a different approach. No import hooks, no codecs,
no operator overloads. Instead, I present a function decorator. Apply
``@patternmatching.transform`` and you're off to the races. Let's take a look:

::

    import patternmatching

    @patternmatching.transform
    def test_demo():
        values = [[1, 2, 3], ('a', 'b', 'c'), 'hello world',
            False, [4, 5, 6], (1, ['a', True, (0,)], 3)]

        for value in values:
            with match(value):
                with 'hello world':
                    print 'Match strings!'
                with False:
                    print 'Match booleans!'
                with [1, 2, 3]:
                    print 'Match lists!'
                with ('a', 'b', 'c'):
                    print 'Match tuples!'
                with [4, 5, quote(temp)]:
                    print 'Bind variables! temp =', temp
                with (1, ['a', True, quote(result)], 3):
                    print 'Nest expressions! result =', result

        print 'Wow, pretty great!'

Finally, pythonic pattern matching! If you've experienced the feature before in
"functional" languages like Erlang, Haskell, Clojure, F#, OCaml, etc. then you
can guess at the semantics. The syntax is an abuse of ``with`` statements. If it
were described in the standard docs, it would look like:

::

    match_stmt  ::= "with" "match" "(" expresssion ")" ["as" name] ":" match_suite
    match_suite ::= NEWLINE INDENT like_stmt+ DEDENT
    like_stmt   ::= "with" expression ":" NEWLINE INDENT suite DEDENT

Hopefully, you noticed the ``match`` and ``quote`` function calls. Those aren't
really function calls and you can change them as needed. ``match`` works by
identifying the ``with`` statement as a match-statement. It would be nice if
Python had a way to define your own types of statements but that's beyond the
scope of this project.

It might have read better if I'd used an ``if`` statement rather than
``with``. In that case you would read:

::

    if match(expression):
        with True:
            print 'The expression is True.'
        with _:
            print 'The expression is not True.'

This has the benefit of sounding better: "if match expression with True ...".
But ``if`` statements don't support the ``as`` syntax. So if you put something
complex inside the ``match`` parens, you can't bind it to a name. You might
instead bind it to a name on the nested ``with`` statements but I'm doing all
this to save keystrokes so that works against the goal.

Now, regarding the ``quote`` function call (that is not really a function call
at all). You can only put the name of a variable within that call and if you do
so it won't be evaluated at runtime. Instead, it will be bound to its matching
value. Imagine it like you're quoting the variable name, but you don't want to
match a string. So to bind a variable you replace ``"my_variable"`` with
``quote(my_variable)``. The former would otherwise match the string
``my_variable``.

It would have been nice if Python supported custom string types for this
purpose. So far I'm aware of normal strings: ``"blah"``, raw strings:
``r"blah"``, unicode strings: ``u"blah"``, and byte strings: ``b"blah"``. When
developing this module, I wished I could create a new kind of ``quoted`` string
which might look like: ``q"blah"``. It's rare that I see a `C++ feature`_ and
look on with envy. This topic more commonly arises when people want a syntax for
stating an ``OrderedDict`` in an expression. Maybe ``OrderedDict{'foo': 20,
'bar': 45}`` is the future. Looks funny today.

.. _`C++ feature`: http://en.wikipedia.org/wiki/C%2B%2B11#User-defined_literals

Caveats
-------

- ``@patternmatching.transform`` must be the inner-most decorator.
- Does not support lambda functions.
- Does not work on nested functions.
- Requires inspect.getsource to work.

Documentation
-------------

- Todo: describe api
- Type-pattern:

::

    import patternmatching, math

    @patternmatching.transform
    def area(shape):
        with match(type(shape)):
            with Triangle:
                return shape.base * shape.height * 0.5
            with Square:
                return shape.side ** 2
            with Rectangle:
                return shape.length * shape.width
            with Circle:
                return math.pi * shape.radius ** 2
            with _:
                raise Exception('unknown shape')

Alternatives
------------

- https://github.com/lihaoyi/macropy
  - module import, but similar design
- https://github.com/Suor/patterns
  - decorator with funky syntax
  - Shared at Python Brazil 2013
- https://github.com/mariusae/match
  - http://monkey.org/~marius/pattern-matching-in-python.html
  - operator overloading
- http://blog.chadselph.com/adding-functional-style-pattern-matching-to-python.html
  - multi-methods
- http://svn.colorstudy.com/home/ianb/recipes/patmatch.py
  - multi-methods
- http://www.artima.com/weblogs/viewpost.jsp?thread=101605
  - the original multi-methods
- http://speak.codebunk.com/post/77084204957/pattern-matching-in-python
  - multi-methods supporting callables
- http://www.aclevername.com/projects/splarnektity/
  - not sure how it works but the syntax leaves a lot to be desired
- https://github.com/martinblech/pyfpm
  - multi-dispatch with string parsing
- https://github.com/jldupont/pyfnc
  - multi-dispatch
- http://www.pyret.org/
  - It's own language
- https://pypi.python.org/pypi/PEAK-Rules
  - generic multi-dispatch style for business rules
- http://home.in.tum.de/~bayerj/patternmatch.py
  - Pattern-object idea (no binding)
- https://github.com/admk/patmat
  - multi-dispatch style

Other Languages
---------------

- https://msdn.microsoft.com/en-us/library/dd547125.aspx F#
- https://doc.rust-lang.org/book/patterns.html Rust
- https://www.haskell.org/tutorial/patterns.html Haskell
- http://erlang.org/doc/reference_manual/expressions.html#pattern Erlang
- https://ocaml.org/learn/tutorials/data_types_and_matching.html Ocaml

Development
-----------

- Requires Python 2.7
- Run ``tox``
- Todo: show translated source code

::

    import patternmatching

    @patternmatching.transform
    def factorial(num):
        with match(num):
            with 1:
                return 1
            with _:
                return num * factorial(num - 1)

TODO
----

- Should this module just be a function like:

::

    def bind(object, expression):
        """Attempt to bind object to expression.
        Expression may contain `bind.name`-style attributes which will bind the
        `name` in the callers context.
        """
        pass # todo

  What if just returned a mapping with the bindings and something
  like bind.result was available to capture the latest expression.
  For nested calls, bind.results could be a stack. Then the `like` function
  call could just return a Like object which `bind` recognized specially.
  Alternately `bind.results` could work using `with` statement to create
  the nested scope.

::

    if bind(r'<a href="(.*)">', text):
        match = bind.result
        print match.groups(1)
    elif bind([bind.name, 0], [5, 0]):
        pass

  Change signature to `bind(object, pattern)` and make a Pattern object. If
  the second argument is not a pattern object, then it is made into one
  (if necessary). Pattern objects should support `__contains__`.

  `bind` could also be a decorator in the style of oh-so-many multi-dispatch
  style pattern matchers.

  To bind anything, use bind.any or bind.__ as a place-filler that does not
  actually bind to values.

- Add like(...) function-call-like thing and support the following:
  like(type(obj)) check isinstance
  like('string') checks regex
  like(... callable ...) applies callable, binds truthy
- Also make `like` composable with `and` and `or`
- Add `when` support somehow and somewhere
- Add __ (two dunders) for place-holder
- Add match(..., fall_through=False) to prevent fall_through
- Use bind.name rather than quote(name)
- Improve debug-ability: write source to temporary file and modify code object
  accordingly. Change co_filename and co_firstlineno to temporary file?
- Support/test Python 2.6, Python 3 and PyPy 2 / 3
- Good paper worth referencing on patterns in Thorn:
  http://hirzels.com/martin/papers/dls12-thorn-patterns.pdf
- Support ellipsis-like syntax to match anything in the rest of the list or
  tuple. Consider using ``quote(*args)`` to mean zero or more elements. Elements
  are bound to args:

::

    match [1, 2, 3, 4]:
        like [1, 2, quote(*args)]:
            print 'args == [3, 4]'

- Match ``set`` expression. Only allow one ``quote`` variable. If present the
  quoted variable must come last.

::

    with match({3, 1, 4, 2}):
        with {1, 2, 4, quote(value)}:
            print 'value == 3'
        with {3, 4, quote(*args)}:
            print 'args = {1, 2}'

- Add "when" clause like:

::

    with match(list_item):
        with like([first, second], first < second):
            print 'ascending'
        with like([first, second], first > second):
            print 'descending'

- Add ``or``/``and`` pattern-matching like:

::

    with match(value):
        with [alpha] or [alpha, beta]:
            pass
        with [1, _, _] and [_, _, 2]:
            pass

- Match ``dict`` expression?
- Match regexp?

Future?
-------

- Provide more generic macro-expansion facilities. Consider if this module
  could instead be written as the following:

::

    def assign(var, value, _globals, _locals):
        exec '{var} = value'.format(var) in _globals, _locals

    @patternmatching.macro
    def match(expr, statements):
        """with match(expr): ... expansion
        with match(value / 5):
            ... statements ...
        ->
        patternmatching.store['temp0'] = value / 5
        try:
            ... statements ...
        except patternmatching.PatternmatchingBreak:
            pass
        """
        symbol[temp] = expand[expr]
        try:
            expand[statements]
        except patternmatching.PatternMatchingBreak:
            pass

    @patternmatching.macro
    def like(expr, statements):
        """with like(expr): ... expansion
        with like(3 + value):
            ... statements ...
        ->
        patternmatching.store['temp1'] = patternmatching.bind(expr, patternmatching.store['temp0'], globals(), locals())
        if patternmatching.store['temp1']:
            for var in patternmatching.store['temp1'][1]:
                assign(var, patternmatching.store['temp1'][1][var], globals(), locals())
            ... statements ...
            raise patternmatching.PatternmatchingBreak
        """
        symbol[result] = patternmatching.bind(expr, symbol[match.temp], globals(), locals())
        if symbol[result]:
            for var in symbol[result][1]:
                assign(var, symbol[result][1][var], globals(), locals())
            expand[statements]
            raise patternmatching.PatternmatchingBreak

    @patternmatching.expand(match, like)
    def test():
        with match('hello' + ' world'):
            with like(1):
                print 'fail'
            with like(False):
                print 'fail'
            with like('hello world'):
                print 'succeed'
            with like(_):
                print 'fail'

I'm not convinced this is better. But it's interesting. I think you could do
nearly this in ``macropy`` if you were willing to organize your code for the
import hook to work.

Project Links
-------------

- `PatternMatching: Python Pattern Matching @ GrantJenks.com`_
- `PatternMatching @ PyPI`_
- `PatternMatching @ Github`_
- `Issue Tracker`_

.. _`PatternMatching: Python Pattern Matching @ GrantJenks.com`: http://www.grantjenks.com/docs/patternmatching/
.. _`PatternMatching @ PyPI`: https://pypi.python.org/pypi/patternmatching
.. _`PatternMatching @ Github`: https://github.com/grantjenks/python-pattern-matching
.. _`Issue Tracker`: https://github.com/grantjenks/python-pattern-matching/issues

Python Pattern Matching License
-------------------------------

Copyright 2016 Grant Jenks

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
