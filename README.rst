xonsh Quickstart
================
xonsh_ is an interactive system shell that is a Python superset. It is
very handy. I'm not going to give you the commercial for it here. If you
know Python and you use Bash ever, you already know why you want more
Python in your shell.

This guide exists because, while the xonsh documentation is well-written
and complete, it involves a lot of reading to be productive with xonsh.
The goal of this guide is to allow you to become productive as quickly
as possible in the interactive prompt, which is accomplished by omitting
a lot of details. Refer to the official documentation for more
information if you need it.

I have another tutorial on `administrative scripting with Python`_ which
is complementary to this one in some ways, in that it shows how to do a
lot of common shell tasks in native Python.

This page will not teach any Python. If you want to use Xonsh and don't
know Python, you're going to need that. Try the `official tutorial`_. I
may at some point write a Python quickstart guide, but I wouldn't hold
your breath. This guide also assumes some familiarity with Bash or other
POSIX shell.

.. contents::

.. _xonsh: https://xon.sh/

.. _administrative scripting with Python:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. _official tutorial: https://docs.python.org/3/tutorial/index.html

A Tale of Two Modes
-------------------
Shell syntax is optimized for launching processes. Python is not. The
xonsh solution is to have two modes.

- xonsh has Python mode and subprocess mode which have different
  syntax.
- Switching between modes is usually implicit, but can be forced_.
- Python mode is just Python with a few extras.
- Subprocess looks a bit like Bash [#]_, but it also has big differences.
- xonsh provides constructs which work across both modes. Understanding
  these constructs is key to effective use of the shell.

Subprocess mode requires a little description, which you will find in
the next section.  Following that, there is a description of the
mechanisms one can use to move data back and forth between these two
modes, and finally some configuration tips.

.. _forced: Substitutions_
.. [#] I say "Bash" frequently in this guide, but I am really referring
  to POSIX shells in general.

Subprocess Mode
---------------
Similarities to POSIX
~~~~~~~~~~~~~~~~~~~~~
Subprocess mode is automatic when a line begins with a name that doesn't
exist in the current scope. In this mode Syntax is superficially similar
to any POSIX shell:

- A list of whitespace-separated arguments is handed to the executable
  as strings.
- these arguments can be quoted to escape special characters and
  whitespace.
- Basic globbing also works, and ``**`` may be used in Python 3.5+ for
  recursive globbing as in some other popular shells.
- ``&&`` and ``||`` work as in a POSIX shell, but the creators encourage
  the use of ``and`` and ``or`` instead.
- Backgrounding processes with ``&`` works. See `job control`_ for more.
- Pipes and I/O redirection also works in a similar way to the shell.
    ``|  >  >>  <  2>``, etc.

Heredocs are not included. xonsh provides additional, more explicite
`redirection syntax`_, but the standard POSIX forms work just fine. I'm
not sure to what extent more advanced use of file descriptors is
supported, but ``2>&1`` (I believe it may be special-cased).

.. code:: sh

  $ ls -l
  total 40
  -rw-r--r-- 1 ninjaaron ninjaaron 26872 Oct  3 23:01 out.html
  -rw-r--r-- 1 ninjaaron ninjaaron 11313 Oct  4 21:32 README.rst
  $ touch 'filename with spaces'
  $ ls -l 'filename with spaces'
  -rw-r--r-- 1 ninjaaron ninjaaron 0 Oct  3 21:15 'filename with spaces'
  $ # here's a glob
  $ echo /usr/*
  /usr/bin /usr/include /usr/lib /usr/lib32 /usr/lib64 /usr/local /usr/sbin /usr/share /usr/src
  $ sudo apt-get update && sudo apt-get dist-upgrade
  [...]
  $ # alternative: sudo apt-get update and sudo apt-get dist-upgrade
  $ firefox &
  $ # firefox is running along on its merry way.
  $ echo /usr/l* | tr a-z A-Z
  /USR/LIB /USR/LIB32 /USR/LIB64 /USR/LOCAL

Differences from POSIX
~~~~~~~~~~~~~~~~~~~~~~
- Strings are Python 3 strings in every way.
- Backslash escapes in arguments are not allowed.
- Strings literals are the only way to escape special shell characters.
  (... excluding so-called `subprocess macros`_...)

.. code:: sh

  $ rm filename\ with\ spaces
  /usr/bin/rm: cannot remove 'filename\': No such file or directory
  /usr/bin/rm: cannot remove 'with\': No such file or directory
  /usr/bin/rm: cannot remove 'spaces': No such file or directory
  $ rm 'filename with spaces'
  $

- No brace expansion yet_ (iterables can be expanded. see: `Python
  Substitution`_)
- quoting part of a string with special characters and leaving another
  part unquoted (perhaps for the use of a glob character or brace
  expansion) is not permitted. The creators of xonsh find this behavior
  to be "insane_".

.. code:: sh

  $ touch "filename with spaces"
  $ ls -l "filename with"*
  /usr/bin/ls: cannot access '"filename with"*': No such file or directory
  $ # ^ someone else's idea of sanity.
  $ # xonsh has additional globbing mechanisms to compensate for this
  $ # lack, which are covered in the next section.

- Command substitution in subprocess mode only works with ``$()``.
  Backticks mean something else in xonsh. Both of these features will be
  covered in more detail in the following section.

That about covers it for the quickstart to subprocesses mode. The next
section deals with passing data between the two modes.

.. _redirection syntax:
  https://xon.sh/tutorial.html#input-output-redirection

.. _subprocess macros:
  https://xon.sh/tutorial_macros.html#subprocess-macros

.. _yet:
  https://github.com/xonsh/xonsh/pull/2868

.. _insane:
  https://xon.sh/tutorial_subproc_strings.html?highlight=insane#the-quotes-stay

.. _job control:
  https://xon.sh/tutorial.html#job-control

Going between the Modes
-----------------------
There are several special xonsh constructs that work both in subprocess
mode and in Python mode which can be useful for carting data around,
though the first feature we'll cover will be globbing, which isn't
exactly a way to move data between the modes.

Globs
~~~~~
aside from the unquoted globbing behavior in subprocess mode, xonsh
supports regex globbing everywhere with backticks. This feels overkill
most of the time, but is extremely useful when you need it. It is also
somewhat necessitated by the omission of brace expansion.

.. code:: sh

  $ echo `/usr/l.*`
  /usr/lib /usr/lib32 /usr/lib64 /usr/local
  $ # in a folder containing folders with dates as names...
  $ ls -d `18\.0[5-6].*`
  18.05.13  18.05.20  18.06.03  18.06.22  18.06.24
  18.05.19  18.05.27  18.06.17  18.06.23
  $ # in Bash this would be `ls -d 18.0{5..6}*`

Likewise, xonsh supports normal globbing syntax everywhere through the
use of g-strings. These are created with backticks and a ``g`` prefix.

.. code:: shell

  $ ls -ld g`/usr/l*`
  drwxr-xr-x 1 root root 137232 Sep 24 20:50 /usr/lib
  drwxr-xr-x 1 root root  38424 Sep 24 20:38 /usr/lib32
  lrwxrwxrwx 1 root root      3 Aug 21 16:21 /usr/lib64 -> lib
  drwxr-xr-x 1 root root     72 Mar 26  2017 /usr/local

This is once again useful for recursive globbing with ``**`` in Python
3.5+.

One very useful feature glob literals in xonsh is that they can be used
to return pathlib.Path_ instances, which are a very pleasant way of
dealing with paths if I do say so myself. This is done by prefixing
either type of glob string with a ``p``

.. code:: bash

  >>> for p in p`/etc/.*`:
  ...     if p.is_dir():
  ...         print(p)
  ...         
  /etc/ImageMagick-6
  /etc/ImageMagick-7
  /etc/NetworkManager
  /etc/UPower
  /etc/X11
  /etc/asciidoc
  /etc/audisp
  /etc/audit
  [...]


.. _pathlib.Path:
  https://docs.python.org/3/library/pathlib.html#basic-use

Environment Variables
~~~~~~~~~~~~~~~~~~~~~
In xonsh, "environment variables" are prefixed with a ``$``, as in Bash.
xonsh's notion of environment variables includes things like ``$HOME``
and ``$PATH``, but also includes the assignment of arbitrary values to
arbitrary names beginning with ``$``, which only exist for the lifetime
of the current shell. These values are global, and they work in both
subprocess mode and Python mode. In subprocess mode, this is how they
are converted into arguments:

- certain built-in environment variables have predefined conversion
  functions, which will create a sensible string representation.
- if a variable doesn't have such a function registered (e.g. any
  variable you create yourself), it will call ``str()`` on the object.

An example of the first kind of variable is ``$PATH`` which is a wrapper
on a list internally, but will print as colon-separated values (as a
``$PATH`` would in Bash).

Environment variables work like any other variable in Python mode. Like
Bash, these variables can be interpolated freely into strings. Unlike
Bash, they don't require quoting for safety.

.. code:: bash

  >>> for $p in p`/etc/.*`:
  ...     if $p.is_dir():
  ...         echo '$p is a directory'
  ...         
  /etc/ImageMagick-6 is a directory
  /etc/ImageMagick-7 is a directory
  /etc/NetworkManager is a directory
  /etc/UPower is a directory
  [...]

Substitutions
~~~~~~~~~~~~~

Python Substitution
+++++++++++++++++++
One problem with user-created environment variables is that they just
call ``str()`` when they are used in subprocess mode. That means:

.. code:: sh

  $ $dirs = ['/usr', '/bin', '/etc']
  $ ls -ld $dirs
  /usr/bin/ls: cannot access '['\''/usr'\'', '\''/bin'\'', '\''/etc'\'']': No such file or directory

The way to get this to do the right thing is with Python substitution.
Python substitution allows embedding the value of arbitrary Python
expressions into commands. If the Python value is an iterable, it will
be split into separate arguments. Python substitution is marked with
``@()``.

.. code:: sh 

  $ dirs = ['/usr', '/bin', '/etc']
  $ ls -ld @(dirs)
  lrwxrwxrwx 1 root root    7 Aug 21 16:21 /bin -> usr/bin
  drwxr-xr-x 1 root root 3068 Sep 25 22:47 /etc
  drwxr-xr-x 1 root root   80 Sep 25 19:43 /usr
  $ echo hello-@('foo    bar     baz'.split())
  hello-foo hello-bar hello-baz
  $ # Cartesian products can also be produced
  $ echo @(list('abc')):@(list('def'))
  a:d a:e a:f b:d b:e b:f c:d c:e c:f

Python substitution only works in subprocess mode (because it is
redundant in Python mode).

Command Substitution(s)
+++++++++++++++++++++++
xonsh has two forms of command substitution. The first is similar to
that of Bash, using ``$()`` syntax.

.. code:: shell
  
  $ ls -l $(which vi)
  lrwxrwxrwx 1 root root 4 Feb 27  2018 /usr/bin/vi -> nvim
  $ # why are permissions on this alias set to 777 instead of 755?
  $ # Oh well...

If this form of substitution is used in Python mode, it returns a
string.

.. code:: sh

  $ print(repr($(which vi)))
  '/usr/bin/vi'

The other form of command substitution only works in Python mode, where
it returns a ``CommandPipeline`` object, which among other things,
implements an iterator that lazily yields lines as they become available
from the process. Trailing newlines are not stripped.

.. code:: python

  >>> for line in !(ls):
  ...     print(line.split())
  ...     
  ['total', '40']
  ['-rw-r--r--', '1', 'ninjaaron', 'ninjaaron', '26872', 'Oct', '3', '23:01', 'out.html']
  ['-rw-r--r--', '1', 'ninjaaron', 'ninjaaron', '10726', 'Oct', '3', '23:20', 'README.rst']

This object has other interesting properties as well. Look at the
documentation_ for further details. This form of substitution is
probably what you generally want in Python mode.

.. _documentation:
  https://xon.sh/tutorial.html#captured-subprocess-with-and

Basic Configuration, etc.
-------------------------
In progress
