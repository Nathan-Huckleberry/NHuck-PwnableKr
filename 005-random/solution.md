# random

```
Daddy, teach me how to use random value in programming!

ssh random@pwnable.kr -p2222 (pw:guest)
```

```
random@ubuntu:~$ cat random.c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

This c file is on the server. We can see it's using the rand function. rand() should be used with srand() to set the seed and generate new numbers.
Without a call to srand(), rand() will use the same seed and generate the same number every time.
Start up gdb and look through main.

```
gdb-peda$ disas main
Dump of assembler code for function main:
   0x00000000004005f4 <+0>:	push   rbp
   0x00000000004005f5 <+1>:	mov    rbp,rsp
   0x00000000004005f8 <+4>:	sub    rsp,0x10
   0x00000000004005fc <+8>:	mov    eax,0x0
=> 0x0000000000400601 <+13>:	call   0x400500 <rand@plt>
   0x0000000000400606 <+18>:	mov    DWORD PTR [rbp-0x4],eax
   0x0000000000400609 <+21>:	mov    DWORD PTR [rbp-0x8],0x0
   0x0000000000400610 <+28>:	mov    eax,0x400760
   0x0000000000400615 <+33>:	lea    rdx,[rbp-0x8]
   0x0000000000400619 <+37>:	mov    rsi,rdx
   0x000000000040061c <+40>:	mov    rdi,rax
   0x000000000040061f <+43>:	mov    eax,0x0
   0x0000000000400624 <+48>:	call   0x4004f0 <__isoc99_scanf@plt>
   0x0000000000400629 <+53>:	mov    eax,DWORD PTR [rbp-0x8]
   0x000000000040062c <+56>:	xor    eax,DWORD PTR [rbp-0x4]
   0x000000000040062f <+59>:	cmp    eax,0xdeadbeef
   0x0000000000400634 <+64>:	jne    0x400656 <main+98>
   0x0000000000400636 <+66>:	mov    edi,0x400763
   0x000000000040063b <+71>:	call   0x4004c0 <puts@plt>
   0x0000000000400640 <+76>:	mov    edi,0x400769
   0x0000000000400645 <+81>:	mov    eax,0x0
   0x000000000040064a <+86>:	call   0x4004d0 <system@plt>
   0x000000000040064f <+91>:	mov    eax,0x0
   0x0000000000400654 <+96>:	jmp    0x400665 <main+113>
   0x0000000000400656 <+98>:	mov    edi,0x400778
   0x000000000040065b <+103>:	call   0x4004c0 <puts@plt>
   0x0000000000400660 <+108>:	mov    eax,0x0
   0x0000000000400665 <+113>:	leave  
   0x0000000000400666 <+114>:	ret    
End of assembler dump.
gdb-peda$ b *0x400601
Breakpoint 3 at 0x400601
```

The return value of rand should be in the %rax register since this is x86-64.

```
[----------------------------------registers-----------------------------------]
RAX: 0x6b8b4567 
RBX: 0x0 
RCX: 0x7ffff7dd10a4 --> 0x16a5bce3991539b1 
RDX: 0x7ffff7dd10a8 --> 0x6774a4cd16a5bce3 
RSI: 0x7fffffffe36c --> 0x6b8b4567 
RDI: 0x7ffff7dd1620 --> 0x7ffff7dd10b4 --> 0x61048c054e508aaa 
RBP: 0x7fffffffe3a0 --> 0x400670 (<__libc_csu_init>:	mov    QWORD PTR [rsp-0x28],rbp)
RSP: 0x7fffffffe390 --> 0x7fffffffe480 --> 0x1 
RIP: 0x400606 (<main+18>:	mov    DWORD PTR [rbp-0x4],eax)
R8 : 0x7ffff7dd10a4 --> 0x16a5bce3991539b1 
R9 : 0x7ffff7dd1120 --> 0x8 
R10: 0x47f 
R11: 0x7ffff7a47f60 (<rand>:	sub    rsp,0x8)
R12: 0x400510 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffe480 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
```

We can see %rax is 0x6b8b4567. So our key needs to be 0xdeadbeef^0x6b8b4567

```
random@ubuntu:~$ python3 -c "print(0xdeadbeef^0x6b8b4567)" | ./random
Good!
Mommy, I thought libc random is unpredictable...
random@ubuntu:~$ 
```
