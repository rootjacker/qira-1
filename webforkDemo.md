This writeup was made with v0.6 of QIRA, install qira-0.6.tar.xz to match exactly. Today we'll be running a little program we wrote ourselves.

# changetest.c #

```
#include <stdio.h>

int main(int argc, char *argv[]) {
  int a = atoi(argv[1])+27;
  printf("got %d\n", a);
  if (a == 37) {
    printf("WINNER\n");
  } else {
    printf("LOSER\n");
  }
}
```

Built with
```
gcc -m32 changetest.c -o changetest  # committed in git as tests/changetest
```

# Starting QIRA #

So if you read the code, it's obvious what the right answer in argv1 is, but let's say we think it's 4.

Run

```
qira ./changetest 4
```

And visit http://localhost:3002/


<table>
<tr>
<td width='600px'><img src='http://wiki.qira.googlecode.com/git/images/webfork/0b.png'></img></td>
<td width='250px'>As you can see in the stdout buffer, it printed LOSER.<br />
Let's scroll up in the instruction dump to find the compare to 37(hex 0x25) which we know must be in the code.<br />For speed, we know it happened soon before the 2nd printf, which we can see in the strace window happened at change <font color='blue'>108</font>. So click that <font color='blue'>108</font> and scroll up.</td></tr>
<tr>
<td width='600px'><img src='http://wiki.qira.googlecode.com/git/images/webfork/1b.png'></img></td>
<td width='250px'>Ahh, found it at change 100.<br />In the memchange viewer, I clicked <font color='#888800'>0xf6fffc4c</font>, because that's where the bad value is.<br />It's 0x1F because 27+4=31 or 0x1F<br />Now to make it 0x25, with the data highlighted, we click <b>Add Change</b>, type "0x25" in the box, press enter.<br />And click <b>fork</b> to run a new fork of the program from the selected change!</td></tr>
<tr>
<td width='600px'><img src='http://wiki.qira.googlecode.com/git/images/webfork/2b.png'></img></td>
<td width='250px'>WINNER<br />I clicked the last change in fork <font color='#555555'>1</font>. Notice the <font color='#555555'>1</font> next to the change number <font color='blue'>138</font>, and that the blue change is in the right vertical timeline.<br />We changed 0x1F to 0x25, 0x25-0x1F=6, we entered 4, and 4+6=10. So next time, run the program with 10.</td></tr>
</table>

# The command prompt output #

![http://wiki.qira.googlecode.com/git/images/webfork/3.png](http://wiki.qira.googlecode.com/git/images/webfork/3.png)