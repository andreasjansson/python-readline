There is a bug in libreadline 6.2. [rl\_sigwinch\_handler](http://git.savannah.gnu.org/cgit/readline.git/tree/signals.c?id=9922d2d7608ea86ff9e42aca0d3342a92748873f#n244) calls [rl\_resize\_terminal](http://git.savannah.gnu.org/cgit/readline.git/tree/terminal.c?id=9922d2d7608ea86ff9e42aca0d3342a92748873f#n351), which calls [\_rl\_get\_screen\_size](http://git.savannah.gnu.org/cgit/readline.git/tree/terminal.c?id=9922d2d7608ea86ff9e42aca0d3342a92748873f#n224), which calls [sh\_set\_lines\_and\_columns](http://git.savannah.gnu.org/cgit/readline.git/tree/shell.c?id=9922d2d7608ea86ff9e42aca0d3342a92748873f#n124), which calls [xmalloc](http://git.savannah.gnu.org/cgit/readline.git/tree/xmalloc.c?id=9922d2d7608ea86ff9e42aca0d3342a92748873f#n56), which calls malloc, which **isn't [reentrant](http://en.wikipedia.org/wiki/Reentrant_%28subroutine%29)**.

One consequence of [nonreentrancy](http://www.gnu.org/software/libc/manual/html_node/Nonreentrancy.html) is that when called in a signal handler, bad things happen. rl\_sigwitch\_handler is a signal handler that gets called on [SIGWINCH](http://en.wikipedia.org/wiki/Unix_signal#SIGWINCH). When you use a window manager like [dwm](http://dwm.suckless.org/), you end up sending SIGWINCH all the time, sometimes in the middle of another malloc.

When this happens, malloc gets stuck in a deadlock in [\_\_lll\_lock\_wait\_private](http://fossies.org/dox/glibc-ports-2.16.0/arm_2nptl_2lowlevellock_8c_source.html#l00025), waiting for a [futex](http://unixhelp.ed.ac.uk/CGI/man-cgi?futex) that never wakes up.

This issue has [already been fixed](http://www.mail-archive.com/bug-readline@gnu.org/msg00319.html) in [a development branch of bash](http://git.savannah.gnu.org/cgit/bash.git/tree/lib/readline/signals.c?h=devel&id=55a5a4acdef700d9fbb84da5e5b5ed7ad11e6f48#n263), but unfortunately that fix hasn't made it upstream readline yet.

My fork of python-readline applies a [patch](https://github.com/andreasjansson/python-readline/blob/master/rl/readline62-005) based on the relevant changes from bash to the signal handling code in libreadline.

Time to go to bed.