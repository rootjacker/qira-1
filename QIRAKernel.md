# Introduction #

Currently, QIRA only operates on user-space programs. The goal of the QIRA kernel project is to allow it to support full systems, a couple things must happen before this can work.

The current instruction limit is too low, the main way to deal with this is better filtering. In the kernel, we should be able to target only one process, and only instrument reads and writes to the processes memory space. This involves the parsing of kernel data structures from a "hypervisor"(this a valid term for QEMU?).

Potentially VT-X could be used for this as well at a massive speedup, then only slow QEMU for the parts in the filter. This is a huge stretch goal, as I don't think interspersing QEMU and KVM has even been done.

QIRA also has no way of nicely marking when an interrupt or context switch happened; you should be able to jump around to these points.

Consider using PANDA to get the logging, then you'll get replay for free. This avoids the syscall problem, and could perhaps one day be the default way to use QIRA.

# Details #

The main target would be Windows, with the end intention of making it usable for my pwn2own attempt next year. I'm not sure how many instructions Windows 8.1 runs to, for example, start up a process, but hopefully it's doable on a beefy machine. The flat database will be written to the hard drive and only small parts would be loaded in to the ram structured db.

The whole system will run inside QEMU, perhaps without TCG though and actually writing in the logging to the generated code.

Also, for the fun of it, let's support 15-410 kernels. I'm using QEMU in singlestep mode, so the interrupts should happen inside basic blocks. I think this would be an awesome debugging tool if done well, and might be a path to solicit some QIRA contributions.