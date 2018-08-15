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

