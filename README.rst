Introduction
============

**IMPORTANT: This project has just started.  The sources here do not contain the changes
listed in this README.rst file yet.**

emtm is the Extended Micro Terminal Multiplexer, a terminal multiplexer based on the
Micro Terminal Multiplexer (mtm) project.

It has a superset of mtm's features and shares its principles to some extent:

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
    Unlikely its predecessor, emtm is an active project that aims to add some extensions to the
    original project, while maintaining its compatibility and avoiding Stability
    disruption.  The goal is to make as few changes as possible to the original
    source code (although this can change in the future if new features really worth it).

    emtm **WILL SUPPORT** the following extensions over mtm:

    - emtm will refuse to run if stdin is not a valid tty device.  This prevents command
      injections from standard input redirection;

    - An option that allows it to work as a wrapper to agetty, allowing it to be
      the lowest level process on a Virtual Terminal. (An selinux policy to be used on Linux is
      provided so that this new functionality works even when selinux is set to enforcing);

    - A systemd "/etc/systemd/system/getty@.service.d/override.conf" file for enabling
      transparent getty wrapping;

    - A loadable keyboard map file for replacing the SHIFT+PGUP and SHIFT+PGDOWN keys with the
      necessary key sequences (i.e., CTRL+g followed by PGUP or PGDOWN) enabling them to work
      on the Linux Virtual Terminals, even if the kernel doesn't report those keystrokes, which is
      still true as of Linux Kernel version 6.11.5;

    - Command line options to show the current emtm version and its usage;

    - A command line option that allows the number of scrollback lines to be specified at emtm
      invocation as opposed to only being configurable at compile time;

    - Some additional envirnoment variables are set within an emtm session:

      - EMTM_TTY: Is set to the root tty name (i.e., the root tty where emtm is running).
        OBS: This variable is only set when the -a option is used;

      - EMTM_TERM: Is set to the root terminal (usually "linux" when run in a Linux VT;
        OBS: This variable is only set when the -a option is used;

      - EMTM_VERSION: Is set to emtm's verision;

      - EMTM_SCROLL: Is set to emtm's number of lines in the scroll back buffer.  If this variable is
        set by the parent process that invocates emtm, the scroll buffer will be set accordingly.
        Further changes in this variable after emtm is already running has no effect on the scroll back
        buffer size;

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

    emtm [-w GETTY] [-v] [-h] [-s LINES] [-T NAME] [-t NAME] [-c KEY]

The `-w` flag tells emtm to execute one of the preconfigured terminal getty
programs to perform login instead of directly starting a shell.
The getty programs and their parameters are are defined in config.h.  By detault
there are four getty options supported:

parameters of the -w option:
    - *a*: Uses agetty;

    - *b*: Uses busybox getty functionality;

    - *m*: Uses mingetty.

    - *M*: Uses mgetty;

OBS: From the options listed above, only agetty directly supports asking /bin/login to propagate
the emtm environment variables to the login session.  It is also possible to propagate them
using mgetty, but this requires additional changes in its configuration files.  The other two
(mingetty and busybox) doesn't support passing args to /bin/login, so the EMTM_... variables
won't be set after the login (/bin/login will clear them before finally invoking the login shell).

The main idea of -w is allow emtm to wrap the actual getty.  In addition to enable the
support of spliting the screen even before login, on Linus it also restores the
Virtual Terminal scrollback functionality which was removed from the Kernel on
versions 5.9 and above.  When this option is used, the environment variable EMTM_ROOT_TTY
is set to the name of the root tty (typically tty1, tty2, tty3, ...) as the tty command will
print the pseudo tty device on the chield terminals (/dev/pts/1, /dev/pts/2, ...);

The `-v` tells emtm to show its version and exit;

The `-h` tells emtm to show its usage and exit;

The `-s` is used to control the number of LINES for the scrollback funcionality at its
invocation time, overriding the SCROLLBACK constant defined in the config.h at compile time.

The `-T` flag tells emtm to assume a different kind of host terminal.

The `-t` flag tells emtm what terminal type to advertise itself as.
Note that this doesn't change how emtm interprets control sequences; it
simply controls what the `TERM` environment variable is set to.

The `-c` flag lets you specify a keyboard character to use as the "command
prefix" for emtm when modified with *control* (see below).  By default,
this is `g`.

Once inside emtm, things pretty much work like any other terminal.  However,
emtm lets you split up the terminal into multiple virtual terminals.

At any given moment, exactly one virtual terminal is *focused*.  It is
to this terminal that keyboad input is sent.  The focused terminal is
indicated by the location of the cursor.

The following commands are recognized in emtm, when preceded by the command
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
(Note that you only need to read this section if you're curious.  emtm should
just work out-of-the-box for you, thanks to the efforts of the various
hackers over the years to make terminal-independence a reality.)

By default, wmtm advertises itself as a `screen-bce` terminal.  This is what 
`GNU screen` and `tmux` advertise themselves as, and is a well-known terminal
type that has been in the default terminfo database for decades.

(Note that this should not be taken to imply that anyone involved in the
`GNU screen` or `tmux` projects endorses or otherwise has anything to do
with emtm, and vice-versa. Their work is excellent, though, and you should
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

