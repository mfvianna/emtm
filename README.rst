Introduction
============

emtm is the Extended Micro Terminal Multiplexer, a terminal multiplexer.

It has four major features/principles:

Simplicity
    There are only a few commands, two of which are hardly ever used.
    There are no modes, no dozens of commands, no crazy feature list.

Compatibility
    emtm emulates a classic ANSI text terminal.  That means it should
    work out of the box on essentially all terminfo/termcap-based systems
    (even pretty old ones), without needing to install a new termcap entry.

Size
    emtm is small.
    The entire project is still around 1000 lines of code.

Stability
    Unlikely mtm, emtm is an active project that aims to add a few extensions to the
    original project, while maintaining its compatibility and avoiding to disrupt the
    Stability as much as possible with just a few changes in the source code.

    emtm **WILL SUPPORT** the following extensions over mtm:

    - An option that allows emtm to work as a wrapper to agetty, allowing it to be
      lowest level process on a Virtual Terminal. (A selinux policy file is provided
      so that it works with selinux enforcing);

    - A systemd "/etc/systemd/system/getty@.service.d/override.conf" file for enabling
      transparent agetty wrapping;

    - A loadable keyboard map for replacing the SHIFT+PGUP and SHIFT+PGDOWN with the
      necessary key sequence enabling them to work on the Virtual Terminals, even if
      the kernel doesn't report those keystrokes, which as of Kernel 6.11.5 is still
      true;

    - Options to show the current emtm version as well as its usage;

    - An option that allows the number of scrollback lines to be specified at emtm
      invocation.


Installation
============
Installation and configuration is fairly simple:

- You need ncursesw.
  If you want to support terminal resizing, ncursesw needs to be
  compiled with its internal SIGWINCH handler; this is true for most
  precompiled distributions.  Other curses implementations might work,
  but have not been tested.
- Edit the variables at the top of the Makefile if you need to
  (you probably don't).
- If you want to change the default keybindings or other compile-time flags,
  copy `config.def.h` to `config.h` and edit the copy. Otherwise the build
  process will use the defaults.
- Run::

    make

  or::

    make CURSESLIB=curses

  or::

    make HEADERS='-DNCURSESW_INCLUDE_H="<ncurses.h>"'

  whichever works for you.
- Run `make install` if desired.

Usage
=====

Usage is simple::

    emtm [-a] [-v] [-h] [-s SCROLL_LINES] [-T NAME] [-t NAME] [-c KEY]

The `-a` flag tells emtm to execute `/sbin/agetty -o "-p -- \\u" - $TERM` instead of
any shell, allowing it to replace (actually wrap) agetty in order to restore the
Virtual Terminal scrollback functionality which was removed from the Linux Kernel from
versions 5.xx.xx and above;

The `-h` tells emtm to show its usage and exit;

The `-v` tells emtm to show its version and exit;

The `-s` <lines>" tells emtm to store a buffer of <lines> for the scrollback funcionality
at its invocation time, overriding the SCROLLBACK constant otherwise confugurable only
through config.h at compile time.

The `-T` flag tells emtm to assume a different kind of host terminal.

The `-t` flag tells emtm what terminal type to advertise itself as.
Note that this doesn't change how mtm interprets control sequences; it
simply controls what the `TERM` environment variable is set to.

The `-c` flag lets you specify a keyboard character to use as the "command
prefix" for mtm when modified with *control* (see below).  By default,
this is `g`.

Once inside emtm, things pretty much work like any other terminal.  However,
mtm lets you split up the terminal into multiple virtual terminals.

At any given moment, exactly one virtual terminal is *focused*.  It is
to this terminal that keyboad input is sent.  The focused terminal is
indicated by the location of the cursor.

The following commands are recognized in mtm, when preceded by the command
prefix (by default *ctrl-g*):

Up/Down/Left/Right Arrow
    Focus the virtual terminal above/below/to the left of/to the right of
    the currently focused terminal.

o
    Focus the previously-focused virtual terminal.

h / v
    Split the focused virtual terminal in half horizontally/vertically,
    creating a new virtual terminal to the right/below.  The new virtual
    terminal is focused.

w
    Delete the focused virtual terminal.  Some other nearby virtual
    terminal will become focused if there are any left.  mtm will exit
    once all virtual terminals are closed.  Virtual terminals will also
    close if the program started inside them exits.

l
    Redraw the screen.

PgUp/PgDown/End
    Scroll the screen back/forward half a screenful, or recenter the
    screen on the actual terminal.

That's it.  There aren't dozens of commands, there are no modes, there's
nothing else to learn.

(Note that these keybindings can be changed at compile time.)

Screenshots
-----------
mtm running three instances of `tine <https://github.com/deadpixi/tine>`_

.. image:: screenshot2.png

mtm running various other programs

.. image:: screenshot.png

mtm showing its compatibility

.. image:: vttest1.png
.. image:: vttest2.png

Compatibility
=============
(Note that you only need to read this section if you're curious.  mtm should
just work out-of-the-box for you, thanks to the efforts of the various
hackers over the years to make terminal-independence a reality.)

By default, mtm advertises itself as a `screen-bce` terminal.  This is what `GNU
screen` and `tmux` advertise themselves as, and is a well-known terminal
type that has been in the default terminfo database for decades.

(Note that this should not be taken to imply that anyone involved in the
`GNU screen` or `tmux` projects endorses or otherwise has anything to do
with mtm, and vice-versa. Their work is excellent, though, and you should
definitely check it out.)

The (optional!) `mtm` Terminal Types
------------------------
mtm comes with a terminfo description file called mtm.ti.  This file
describes all of the features supported by mtm.

If you want to install this terminal type, use the `tic` compiler that
comes with ncurses::

    tic -s -x mtm.ti

or simply::

    make install-terminfo

This will install the following terminal types:

mtm
    This terminal type supports all of the features of mtm, but with
    the default 8 "ANSI" colors only.

mtm-256color
    Note that mtm is not magic and cannot actually display more colors
    than the host terminal supports.

mtm-noutf
    This terminal type supports everything the mtm terminal type does,
    but does not advertise UTF8 capability.

That command will compile and install the terminfo entry.  After doing so,
calling mtm with `-t mtm`::

    emtm -t mtm

will instruct programs to use that terminfo entry.
You can, of course, replace `mtm` with any of the other above terminal
types.

Using these terminfo entries allows programs to use the full power of mtm's
terminal emulation, but it is entirely optional. A primary design goal
of mtm was for it to be completely usable on systems that didn't have the
mtm terminfo entry installed. By default, mtm advertises itself as the
widely-available `screen-bce` terminal type.

Copyright and License
=====================

Copyright 2016-2019 Rob King <jking@deadpixi.com>
Copyright 2024 Marcelo Vianna <<TODO>>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

