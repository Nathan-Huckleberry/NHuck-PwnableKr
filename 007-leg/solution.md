# leg

```
Daddy told me I should study arm.
But I prefer to study my leg!

Download : http://pwnable.kr/bin/leg.c
Download : http://pwnable.kr/bin/leg.asm

ssh leg@pwnable.kr -p2222 (pw:guest)
```

Looks like we just need to determine values of key1, key2 and key3 for the flag.

```
 int main(){
      int key=0;
      printf("Daddy has very strong arm! : ");
      scanf("%d", &key);
      if( (key1()+key2()+key3()) == key ){
          printf("Congratz!\n");
          int fd = open("flag", O_RDONLY);
          char buf[100];
          int r = read(fd, buf, 100);
          write(0, buf, r);
      }
      else{
          printf("I have strong leg :P\n");
      }
      return 0;
  }
```

Disassembled functions.

```
(gdb) disass key1                                                                                  
Dump of assembler code for function key1:                                                          
   0x00008cd4 <+0>: push    {r11}       ; (str r11, [sp, #-4]!)                                    
   0x00008cd8 <+4>: add r11, sp, #0                                                                
   0x00008cdc <+8>: mov r3, pc                                                                     
   0x00008ce0 <+12>:    mov r0, r3                                                                 
   0x00008ce4 <+16>:    sub sp, r11, #0                                                            
   0x00008ce8 <+20>:    pop {r11}       ; (ldr r11, [sp], #4)                                      
   0x00008cec <+24>:    bx  lr                                                                     
End of assembler dump.                                                                             
(gdb) disass key2                                                                                  
Dump of assembler code for function key2:                                                          
   0x00008cf0 <+0>: push    {r11}       ; (str r11, [sp, #-4]!)                                    
   0x00008cf4 <+4>: add r11, sp, #0                                                                
   0x00008cf8 <+8>: push    {r6}        ; (str r6, [sp, #-4]!)                                     
   0x00008cfc <+12>:    add r6, pc, #1                                                             
   0x00008d00 <+16>:    bx  r6                                                                     
   0x00008d04 <+20>:    mov r3, pc                                                                 
   0x00008d06 <+22>:    adds    r3, #4                                                             
   0x00008d08 <+24>:    push    {r3}                                                               
   0x00008d0a <+26>:    pop {pc}                                                                   
   0x00008d0c <+28>:    pop {r6}        ; (ldr r6, [sp], #4)                                       
   0x00008d10 <+32>:    mov r0, r3                                                                 
   0x00008d14 <+36>:    sub sp, r11, #0                                                            
   0x00008d18 <+40>:    pop {r11}       ; (ldr r11, [sp], #4)                                      
   0x00008d1c <+44>:    bx  lr                                                                     
End of assembler dump.                                                                             
(gdb) disass key3                                                                                  
Dump of assembler code for function key3:                                                          
   0x00008d20 <+0>: push    {r11}       ; (str r11, [sp, #-4]!)                                    
   0x00008d24 <+4>: add r11, sp, #0                                                                
   0x00008d28 <+8>: mov r3, lr                                                                     
   0x00008d2c <+12>:    mov r0, r3                                                                 
   0x00008d30 <+16>:    sub sp, r11, #0                                                            
   0x00008d34 <+20>:    pop {r11}       ; (ldr r11, [sp], #4)                                      
   0x00008d38 <+24>:    bx  lr                                                                     
End of assembler dump. 
```

The first function looks like it just returns the program counter. 0x8ce0
The second function looks like it returns the program counter + 4. 0x8d0a
The third function returns lr. lr in arm is the return address. 0x8d80

0x8d80 + 0x8d0a + 0x8ce0 = 108394

```
/ $ ./leg
Daddy has very strong arm! : 108394
I have strong leg :P
```

My solution didn't work. :( I have no experience with arm so there's probably an issue there. I'm pretty sure I'm close and am just missing something ARM specific so I'm just going to bruteforce.

```
from pwn import *;

s = ssh(host='pwnable.kr',
        port=2222,
        user='leg',
        password='guest');
  
num = 108394
shell = s.shell()
sleep(5)
shell.recv();

for i in range(200):
    shell.sendline('./leg')
    shell.sendline(str(num+i))
    sleep(1)
    print(shell.recv().decode('ascii'))
shell.interactive()
```

```
./leg
Daddy has very strong arm! : 108394
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108395
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108396
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108397
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108398
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108399
I have strong leg :P
/ $ 
./leg
Daddy has very strong arm! : 108400
Congratz!
My daddy has a lot of ARMv5te muscle!
```

Looks like I was 6 off for some reason?
