<h1>flag</h1>

```
Papa brought me a packed present! let's open it.

Download : http://pwnable.kr/bin/flag

This is reversing task. all you need is binary
```

I started by running the code just to see what happened. Seems like it should be easy enough at first sight.

```
(ctf) [huck@knuth] ./flag                  
I will malloc() and strcpy the flag there. take it.
```

Unfortunately gdb doesn't work because the binary has been stripped

```
gdb-peda$ disas main
No symbol table is loaded.  Use the "file" command.
```

The challenge statement says `packed present`. This sounds like the code was packed.

```
(ctf) [huck@knuth] strings flag | grep pack -
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
```

Seems like UPX was used to pack

```
(ctf) [huck@knuth] upx -d flag 
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2013
UPX 3.91        Markus Oberhumer, Laszlo Molnar & John Reiser   Sep 30th 2013

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    887219 <-    335288   37.79%  linux/ElfAMD   flag

Unpacked 1 file.
```

After decompressing the challenge is pretty straightforward. The flag string is stored at 0x6c2070.

```
gdb-peda$ disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   rbp
   0x0000000000401165 <+1>:	mov    rbp,rsp
   0x0000000000401168 <+4>:	sub    rsp,0x10
   0x000000000040116c <+8>:	mov    edi,0x496658
   0x0000000000401171 <+13>:	call   0x402080 <puts>
   0x0000000000401176 <+18>:	mov    edi,0x64
   0x000000000040117b <+23>:	call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:	mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:	mov    rsi,rdx
   0x0000000000401192 <+46>:	mov    rdi,rax
   0x0000000000401195 <+49>:	call   0x400320
   0x000000000040119a <+54>:	mov    eax,0x0
   0x000000000040119f <+59>:	leave  
   0x00000000004011a0 <+60>:	ret    
End of assembler dump.
gdb-peda$ x/s *0x6c2070
0x496628:	"UPX...? sounds like a delivery service :)"
```
