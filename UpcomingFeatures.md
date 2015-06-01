# What's Next #
  * dynamic program slicing
  * qemu system support, with better filters.
  * wine?
  * vim plugin for source code exploration
  * translation to llvm, and symbolic execution

# Short term(Phase 2, this week) #

Support for multiple runs, rewinding, and forking of execution state. Manual exploration of codepaths. qira-server manages execution of the program. Rewinding already works in trunk.

Real fork() calls should behave like this, test on server CTF problems.

![http://wiki.qira.googlecode.com/git/images/features/forks.png](http://wiki.qira.googlecode.com/git/images/features/forks.png)

# Long term(Phase 3, starting next week) #

Automatic exploration of codepaths with symbolic execution. Using something like KLEE.

# Dealing with large binaries #

Bring the binary up without logging in QEMU, then use CRIU to checkpoint the whole QEMU process. Changelist 0 doesn't have to be the ELF, it can be the ELF plus a base set

Dealing with large numbers of changes in general will probably require moving out of Python. Who wants to write(or adopt an existing) C++ database?

# Other Ideas #

  * You want analysis like taint tracking. Write it!
    * http://en.wikipedia.org/wiki/Program_slicing
  * You want nicer looking clickable assembly. Write it!
    * https://code.google.com/p/distorm/
  * You want a custom awesome CFG view. Write it!
    * There is some stuff in old/analysis

# Scratchpad #
  * Save the old changes from the forks?
  * Annotate stdout with which change did it, how to do this?
  * Show the changes in the hex editor, also show the location of registers(harder with strings)

# Old CFG View #
![http://wiki.qira.googlecode.com/git/images/features/oldcfg.png](http://wiki.qira.googlecode.com/git/images/features/oldcfg.png)