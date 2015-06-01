# Introduction #

So in QIRA 1.0, the focus was on dynamic execution only, simply being a trace viewer. If my internship is extended, my plan is to start pulling in a static half.

This was chosen over symbolic execution for now because it's more likely to show results in the short term, and is sort of a prereq for the end dream of symbolic exec.

When this is in a good state, qira ./a.out will no longer run the binary by default.

Since we are heading to CMU, BAP will be the first static backend instead of IDA as stated below.

# Details #

Currently QIRA has two pieces of static, the IDA plugin and CDA. The plan is to unify all the static data from different sources, and display them alongside the current trace.

IDA will be a backend for QIRA, used as a way to statically analyse a binary. It will generate function boundaries, basic block boundaries, disassembly, and xrefs. Basically, all we want here is the IDA kernel. CFGs will be generated and laid out by QIRA.

The big win, and something I've been wanting but never seen done well, is a way to view source code interspersed with assembly. Tons of the subtle type integer overflow bugs in libpng or ffmpeg are actually easier to see in assembly, but easier to navigate to in source. IDA's xrefs are better than any xrefs in an IDE I've seen, using them to jump around source would be a win.

With DWARF, we may be able to remove the clang ast generator as well. DWARF has a lot of info that isn't really viewed well anywhere, I want to dive deep into it and get it all. Make lists of all the things you want highlighted in C++, and try to get them from DWARF. DWARF's big win is that it's already there, and usually very easy to generate on large codebases, including Chrome and the Linux Kernel.

This will finally be the right tool for the stkof solve, and hopefully expand the main use case of QIRA beyond CTFs and exploitation to manual analysis to find bugs in real codebases.

A goal of QIRAStatic is for people to never use vim and grep over static codebases again. Enumerate all the use cases and build a strict superset.

# Moving analysis into another process? #

The QIRA api should include everything required to move analysis out to a second process. Though if it's Python, we'll have the same server issues, like we can't have a server thread and things operating on the data.

What we can do however, is push analysis stuff to the main QIRA process every once in a while, and have it be the only server.

Perhaps scrollables should be factored out in the javascript, idump, strace, procmaps. We should also have a bit of a view manager, ugh.

# Removing base, env, strace, and asm #

base could be turned into big changes in the trace file, I don't think this is appropriate.

env could be turned into a big change at cl 0, I approve of this.

strace doesn't need to exist at all, the info to reconstruct this lives in the trace. I like this idea, but where to move the mmap hack? Perhaps add dim flags for every syscall.

asm is a cute hack, but fetching the memory and disassembling on the fly as needed is much nicer. This solves the JIT problem as well.

# IDB Reading. Writing? #

This would just be a fun project, and give inspiration for all the different types of data to store.

# Using IDA processor modules #

Is this possible?