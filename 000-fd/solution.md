<h1>fd</h1>
<p>The challenge greets us with</p>

```
Mommy! what is a file descriptor in Linux?

try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
https://youtu.be/971eZhMHQQw

ssh fd@pwnable.kr -p2222 (pw:guest)
```
After ssh to the server we can see there is a file called fd.c
![Alt text](image.png?raw=true)

We can input a file descriptor for the input reading. We want to use standard in so we need to get fd=0. If we input 0x1234 (4660) we should be able to input whatever we want.

![Alt text](image2.png?raw=true)
