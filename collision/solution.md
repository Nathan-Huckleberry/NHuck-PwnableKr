The challenge starts with

```
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)
```

Seems like we need to make a hash collision

[Alt text](image.png?raw=true)

The hash function is the sum of the 5 bytes as integers. 

```
./col $(python3 -c "import os; os.write(1, b'\xe8\x01\x01\x01\x01\x05\x01\x01\x01\x01\xd9\x01\x01\x01\x01\x1d\x01\x01\x01\x01')")
```

Each integer is read in little endian and \x00 marks the end of a string so we need to use something other than \x00 as filler.
I just used \x01 and subtracted 4 from each value. I'm using os.write instead of print because python3 print() automatically encodes instead of printing raw binary.

[Alt text](image2.png?raw=true)
