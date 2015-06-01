# Shipping Product #

A sub 2mb, Ubuntu 12.04-14.04, 64-bit best, 32-bit supported, QIRA. Should install on a fresh vm with ./install.sh. ./qemu\_build.sh must work in RELEASE.

## ~~Performance~~ ##

QEMU writes the traces about 10x-100x faster than qiradb can read them. I think this is acceptable.

Unacceptable is any noticeable lag in the UI. It takes a noticeable amount of time sometimes after I click a change for it to update. This issue doesn't appear to be on the backend, as if any socket calls take longer than 20ms, they print a message.

Flag drawing is a known source of slowness. The current LIMIT in the backend is 50(reduced from 200), and I'm still seeing issues. Currently, flags are divs, perhaps if this is all redone with canvas it would be fast. LIMIT is an issue in and of itself, in that it makes things hard to see sometimes. LIMIT must be addressed before shipping v1.

The analyzer is also slow, and currently runs in the same thread as the webserver. Not sure how much we can do about this, aside from adding a timeout to the analyzer or something. I mean, we could move it into C++ I guess, but only if we 100% like it, and picture drawing would still be slow.

Before we can ship 1.0, all perf issues must be hammered out.

## ~~UI / History~~ ##

Should we make the window resizable? Little things like hex editor jitter must be gone, and test this for 64-bit pointers.

The back button must always do something sane. The current iaddr/clnum update is bad, as well as scrolling pushing an entry to the history.

Clicking must behave as expected based on color. Red iaddrs in idump must behave like red iaddrs in hexdump.

## ~~Javascript 64-bit~~ ##

There are places in the javascript where numbers are still used. This needs to be addressed, probably with a library that operates on canonical hex strings.

To be fair, we haven't seen this cause problems lately, but it's so deeply broken that I really shouldn't try to weasel out of it.

Done, test it much?

## ~~mmap backed files~~ ##

Like libraries. Must appear in the hexdump view, read the strace and look for file backed mmaps.

This also should support reading in the DWARF cache if it exists. CDA should be rethought a bit here, remember 1.0 is blocking on stkof easy solution.

Okay, so CDA doesn't work, but we are printing the hash, and that's good enough.

## ~~Library Filtering~~ ##

The current hack seems to work pretty well, but I'm not so sure it's shippable on release. Consider a subset of QIRA filter language, both PIN and QEMU must support it.

Pushed to after v1, --gate-trace added for now.

## ~~Architecture Support~~ ##

  * QEMU and PIN: x86, x86-64
  * QEMU only: arm, aarch64, ppc

Nuance in arches must be hammered out. aarch64 has wrong registers.

x86 and x86-64 will rely on the system's libraries, all others should be fetched with ./fetchlibs.sh(perhaps run by default, or y/n gated in install). Nah, add to install instructions, it's fairly orthogonal.

Analyzer shouldn't detect branches how it does, this is arch specific, at least move it to the qira\_program arch stuff.

## ~~PIN~~ ##

This should ship as an out of the box always built feature. I'm assuming PIN uses ptrace on the real binary, so the loader stuff isn't hacked up by QEMU, and it perfectly matches a normal run of the binary, ASLR and all. If this is not the case, we need to write a ptrace backend.

  * Must stream updates in real time, not just when the program exits.
  * Issues with fork naming and left right must be solved.
  * Clipping in the fork box isn't acceptable. Names too long?
  * x86-64 registers are wrong.

Capstone is the disassembler here, make sure it gets installed.

There are still minor issues here, like that the syscall data isn't loaded except for reads, but it's good enough for now. This will also remain a separate install script, since install really should work super out of the box for everyone.

## ~~CDA and DWARF~~ ##

Must work out of the box. Write a test for this. Though this is still considered a beta feature, if the C analysis stuff isn't perfect that's okay, as long as it has some semblance of working. Address the --dwarf, --cda, and --cda-only flag issue, and replace with a single --cda flag. Perhaps cda should even be the default if there is DWARF data?

Consider what else we can do with DWARF data as well. Don't add features, but if it's already there and should work, make it work. Do not violate expectations.

Testing isn't done for this yet, but that's really in testing.

## ~~IDA Plugin~~ ##

There's snapback issues with scroll, and IDA has issues with stealing the focus. These things really hamper usability.

## ~~Windows~~ ##

QIRA must run and work on Windows using PIN. Windows IDA plugin needs a build script, and must be kept in sync with the others.

Can we run PIN inside of WINE?

Working on Windows has been pushed to a later release, but the Windows IDA plugin is being built well, so good enough.

## ~~buildbot and testing~~ ##

Ideally, QIRA should be built in a consistent VM. We are shipping QEMU binaries and IDA plugin binaries, though building the IDA plugin for the three platforms isn't easy.

Something needs to be done about testing.

Eh, we'll call mingw a win here, tested install in ubuntu and fedora.

## ~~webforking~~ ##

This is a nongoal for v1.0. We should remove support in the shipping version and gate it behind a hidden flag, it's just too hard to make it work always as people expect, and there's little room for debugging.

## ~~strace~~ ##

  * QEMU -- Fix the printing of addresses, upstream this into real QEMU?
  * PIN -- Clean it up with real names and argument count.

Fixed enough to be usable.

## Target Test Programs ##

In addition to all the stuff qira works well on, let's gate on these 4 problems that it didn't work well on before we can ship 1.0.

  * **fixed by adding --gate-trace** defcon problem(eliza) had a big init thing I needed to skip(QIRA filter language?, also needed for things like chrome support, which should work in qira pin)
  * **fixed with proper aarch64 support** hitcon had an aarch64 problem I forget the name of(aarch64 support, half added during ctf)
  * **nice cda will have to wait, but works with qemu well enough, i was stupid during ctf** hitcon had a big heap overflow(stkof) into TLS(qira pin should have worked here, didn't try, switched to gdb when qira qemu failed and this problem took me 4 hours vs probably 1 with a working QIRA, cost me top 5). the right solution here is using CDA on the libc, make it work.
  * **pushed to later release** hitcon had a windows binary(hop), wasn't sure about the qira windows support so used windbg(probably a 30-minute cost)