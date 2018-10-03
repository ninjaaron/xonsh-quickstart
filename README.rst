xonsh Quickstart
================
xonsh_ is a shell language like Bash or PowerShell or whatever that is a
Python superset. It is very handy. I'm not going to give you the
commercial for it here. If you know Python and you use Bash ever, you
already know why you want more Python in your shell.

This guide exists because, while the xonsh documentation is well-written
and complete, it involves a lot of reading to get enough understanding
to be productive with xonsh. The goal of this guide is to allow you to
become productive as quickly as possible in the interactive prompt,
which is accomplished by omitting a lot of details. Refer to the
official documentation for more information if you need it.

This page will also not teach any Python. If you want to use Xonsh and
don't know Python, you're going to need that. Try the `official
tutorial`_. I may at some point write a Python  uickstart guide, but I
wouldn't hold your breath.

Also, one warning: It's a bad idea to set xonsh as your default shell
with ``chsh``. While this isn't a problem most of the time, there are
occasionally applications which will "shell out" to whatever you user's
shell is. They are expecting a POSIX shell. xonsh is very likely to
wreck their world. Instead, go into your terminal preferences and set
xonsh as the starup command, or make a keybinding to launch a terminal
running xonsh.

.. contents::

.. _xonsh: https://xon.sh/
.. _official tutorial: https://docs.python.org/3/tutorial/index.html

A Tale of Two Modes
-------------------
You already know the upside of xonsh: It's Python and not Bash. Bash is
a decent interactive shell and a rather poor programming language.

Here's the downside of xonsh: shells are optimized for interactively
starting processes. Python is not (to say the least!). The xonsh
compromise, and I think it may be the right one, is to have two modes:
Python mode, where everything is (basically) Python, and subprocess
mode, where things look and act a lot more like Bash (without all the
dangerous spiky edges).

These two modes are essentially what allows to xonsh to be an effective
shell and a Python superset. It also means you're essentially using two
different languages in the shell. These modes are switched between
implicitly in certain contexts. Python mode is essentially Python, so
it will not receive a section with its own explanation. However, command
mode requires a little description, which you will find in the next
section. Following that, there is a description of the mechanisms one
can use to move data back and forth between these two modes.

Command Mode
------------
Command mode is entered automatically when a line begins with a name
that doesn't exist in the current scope. In this mode, the syntax is
similar to any POSIX shell (including Bash). The name of the executable
is followed by a sequence of arguments split on whitespace, which are
interpreted as an array of strings in the program being executed. As in
the shell, these strings can be quoted to escape special
characters. This means most commands will look very similar to the way
they would in any other shell. Basic globbing also works, and ``**`` may
be used for recursive globbing as in some other shell dialects. Pipes
and I/O redirection also works in a similar way to the shell, though
heredocs are not included. Though xonsh provides additional `redirection
syntax`_, the standard mechanisms work just fine.

.. code:: sh

  $ ls -l /usr
  total 8
  drwxr-xr-x 1 root root  63194 Sep 25 19:43 bin
  drwxr-xr-x 1 root root  20842 Sep 24 20:38 include
  drwxr-xr-x 1 root root 137232 Sep 24 20:50 lib
  drwxr-xr-x 1 root root  38424 Sep 24 20:38 lib32
  lrwxrwxrwx 1 root root      3 Aug 21 16:21 lib64 -> lib
  drwxr-xr-x 1 root root     72 Mar 26  2017 local
  lrwxrwxrwx 1 root root      3 Aug 21 16:21 sbin -> bin
  drwxr-xr-x 1 root root   3352 Sep 16 14:52 share
  drwxr-xr-x 1 root root      0 Mar 26  2017 src
  $ touch 'filename with spaces'
  $ ls -l 'filename with spaces'
  -rw-r--r-- 1 ninjaaron ninjaaron 0 Oct  3 21:15 'filename with spaces'
  $ # here's a glob
  $ echo /usr/*
  /usr/bin /usr/include /usr/lib /usr/lib32 /usr/lib64 /usr/local /usr/sbin /usr/share /usr/src
  $ echo /usr/l* | tr a-z A-Z
  /USR/LIB /USR/LIB32 /USR/LIB64 /USR/LOCAL

No big surprises there. However, there are major differences from a
POSIX shell dialect. For one thing, you may be glad to learn that the
strings are Python 3 strings in every way. They can take the same kinds
of prefixes Python strings can (bytes strings also work), use the same
escape sequences, same rules for double and single quotes, and
triple-quote strings are also allowed. In Python 3.6, f-strings also
work.

Some things are also missing. The one I miss the most is brace
expansion, though xonsh does offer array expansion, as we will see in
the following section. Additionally, quoting part of a string with
special characters and leaving another part unquoted (perhaps for the
use of a glob character) is not permitted. The creators of xonsh find
this behavior to be "insane_". I find its omission to be rather
annoying, and the command-mode way of interpreting such strings is not
an improvement by any stretch. In any case, xonsh has mechanisms to
compensate for some of this which will be covered in the next section.

Command mode also supports ``&&`` and ``||`` operators for running
additional commands on success or failure, However, they recommend using
the more Pythonic-looking ``and`` and ``or`` operators.

Backgrounding processes with ``&`` also works. See `job control`_ for
more.

Command substitution in command mode only works with ``$()``. Backticks
mean something else in xonsh. Both of these features will be covered in
more detail in the following section.

That about covers it for the quickstart to command mode. The next
section deals with passing data between the two modes.

.. _redirection syntax:
  https://xon.sh/tutorial.html#input-output-redirection

.. _insane:
  https://xon.sh/tutorial_subproc_strings.html?highlight=insane#the-quotes-stay

.. _job control:
  https://xon.sh/tutorial.html#job-control

Going between the Modes
-----------------------
There are several special xonsh constructs that work both in command
mode and in Python mode which can be useful for carting data around,
though the first feature we'll cover will be globbing, which isn't
exactly a way to move data between the modes.

Globs
~~~~~
aside from the unquoted globbing behavior in command mode, xonsh
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

This is once again useful for recursive globbing with ``**``.

One very useful feature about globs is that they can be used to return
pathlib.Path_ instances, which are a very pleasant way of dealing with
paths if I do say so myself. This is done by prefixing either type of
glob string with a ``p``

.. code:: python

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
and ``$SHELL``, but also includes the assignment of arbitrary values to
arbitrary names beginning with ``$``, which only exist for the lifetime
of the current shell. These values are global, and they work in both
command mode and Python mode. In command mode, there values will have
``str()`` called on them when they are converted into arguments, but
they work like any other variable in Bash. Like Bash, these variables
can be interpolated freely into strings. Unlike Bash, they don't require
quoting for safety.

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
One problem with environment variables is that they just call ``str()``
when they are used in command mode that means:

.. code:: sh

  $ $dirs = ['/usr', '/bin', '/etc']
  $ ls -ld $dirs
  /usr/bin/ls: cannot access '['\''/usr'\'', '\''/bin'\'', '\''/etc'\'']': No such file or directory

The way to get this to do the right thing is with Python substitution.
Python substitution allows embedding the value of arbitrary Python
expressions into commands. If the Python value is an iterable, it will
be split into separate arguments. Python interpolation is marked with
``@()``.

.. code:: sh 

  $ dirs = ['/usr', '/bin', '/etc']
  $ ls -ld @(dirs)
  lrwxrwxrwx 1 root root    7 Aug 21 16:21 /bin -> usr/bin
  drwxr-xr-x 1 root root 3068 Sep 25 22:47 /etc
  drwxr-xr-x 1 root root   80 Sep 25 19:43 /usr
  $ echo @('foo    bar     baz'.split())
  foo bar baz

Python substitution only works in command mode (because it is redundant
in Python mode).

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
