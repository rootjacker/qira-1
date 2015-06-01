An open source program to do dynamic analysis as well as IDA does static analysis.

Think of QIRA as a "competitor" to strace and gdb.

## ##### New Home ##### ##

Development continues at https://github.com/BinaryAnalysisPlatform/qira

Thanks so much to Google for getting me started on this, now it's off to CMU for QIRA phase 2.

For now, we'll leave this page up to document QIRA development until v1.

The wiki may also continue to be used, though this source tree will be out of date.

## Requirements ##
  * A Linux machine, preferably 64-bit Ubuntu.

## Installation ##
```
cd ~/
wget -qO- https://qira.googlecode.com/git/releases/qira-1.0.tar.xz | unxz | tar x
cd qira
./install.sh
```

Optional commands
```
./fetchlibs.sh   # fetch libraries for ppc, armhf, armel, and aarch64
./pin_build.sh   # install PIN plugin for QIRA
./cda_build.sh   # install CDA
```

## Removal ##
```
rm -rf ~/qira /tmp/qira*
sudo rm /usr/local/bin/qira /usr/local/bin/cda
sudo pip uninstall qiradb
```

## Usage ##
Preface the process you want to analyze with qira
```
qira /bin/ls /          # and for arm, QEMU_LD_PREFIX works, or run fetchlibs.sh
```

Navigate to http://localhost:3002/ and view.

To use in server mode(like socat)
```
qira -s tests/ctf/ezhp
nc localhost 4000       # web triggered forks listen on port 4001
```

If you have IDA running with the plugin on the web browser machine it should connect

## Limitations ##
  * Traces longer than 10,000,000 instructions don't work well
  * Web forks only work on i386

## Changelog ##
  * v1.0 -- Perf is good! Tons of bugfixes. Quality software. http://qira.me/
  * v0.9 -- Function indentation. haddrline added(look familiar?). Register highlighting in hexdump.
  * v0.8 -- Intel syntax! Shipping CDA(cda a.out) and experimental PIN backend. Bugfixes. Windows support?
  * v0.7 -- DWARF support. Builds QEMU if distributed binaries don't work. Windows IDA plugin.
  * v0.6 -- Added changes before webforking. Highlight strace addresses. Default on analysis.
  * v0.5 -- Fixed regression in C++ database causing wrong values. Added PowerPC support. Added "A" button.
  * v0.4 -- Using 50x faster C++ database. strace support. argv and envp are there.
  * v0.3 -- Built in socat, multiple traces, forks(experimental). Somewhat working x86-64 and ARM support
  * v0.2 -- Removed dependency on mongodb, much faster. IDA plugin fixes, Mac version.
  * v0.1 -- Initial release

## FAQ ##
**Is IDA needed to use this?**

No. Think of IDA like a joystick for a flight simulator or Push for Ableton. It helps you navigate around, but if you can navigate a program in gdb without IDA, you can navigate in QIRA without IDA.

**Why would I use this? I can do this all in gdb.**

Why would you use IDA when you can use objdump, a printer, scissors, and a gluestick?

**QIRA is broken, fix it fix it fix it fix it**

Please try following the Removal and Installation instructions above, i.e. reinstall, i.e. unplug it and plug it back in. If it still doesn't work, file a bug.

## Thanks ##
Google and Project Zero for the internship and support. QEMU for having clean readable code, and meteor for moving js from the stone ages, albeit slowly.

The EDA and CDA projects. clockish/percontation for the PIN backend. Everyone who refactors my code, contributes bugfixes, and contributes features. Keep contributing!

And the authors of python, llvm/clang, codesearch, capstone, pin, all the pip modules I pull in, and every other open source project this relies on that I can't think of off the top of my head.