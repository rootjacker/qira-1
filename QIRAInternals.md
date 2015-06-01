# Execution trace at /tmp/qira\_logs/<trace number> #

These are written by the modified QEMU. First change at +0x18

```
struct change {
  uint64_t address;
  uint64_t data;
  uint32_t changelist_number;
  uint32_t flags;
};
```

flags

```
#define IS_VALID      0x80000000
#define IS_WRITE      0x40000000
#define IS_MEM        0x20000000
#define IS_START      0x10000000
#define IS_SYSCALL    0x08000000
#define SIZE_MASK 0xFF
```

0x18 file header(changes start at 1)

```
struct logstate {
  uint32_t change_count;
  uint32_t changelist_number;
  uint32_t is_filtered;
  uint32_t first_changelist_number;
  int parent_id;
  int this_pid;
};
```

See middleware/qira\_log.py for reading library

The tci(interpreted QEMU) was modified to output these, see qemu\_mods/tci.c

# QIRA Basic Block #

Loops and functions are detectable dynamically without any knowledge of the underlying instruction set. See old/analysis

The definition of a QIRA basic block:
> A contiguous set of instructions such that every instruction except the first one is always run right after the same instruction and every instruction except the last one is always run right before the same instruction. realblocks {start: 0x800400, end:0x800416, icount: 4, idx:17}

The definition of a QIRA trace:
> An array of indexes of QIRA basic blocks representing a graph.

```
[0, 1, 2, 3, 4, 5, 2, 6, 4, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 16, 15, 17, 18, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 20, 19, 21, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 23, 22, 24, 25]
```

# CSS in QIRA #
  * addr -- this is an address, where the innerHTML should show 0x... but could show a name
    * these are made by highlight, registers, or hexdump
  * addr\_0x1234 -- every address tag has this, for highlighting and naming