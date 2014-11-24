Title: CSCamp 2014 - ELF 1 [Reversing100]
Date: 2014-11-23 22:00
Tags: cscamp, 2014, reversing, 100
Category: reversing
Slug: cscamp-2014-elf1
Author: dynadolos

In this challenge, we were given a file (binary) with no additional instructions provided.

To get a better idea of what we're dealing with, we run the `file` command to get more details about the provided file.

```
dynadolos@dystopia$ file binary
binary: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=215adc183a31f0c6950f8473758a329124e6c1db, not stripped
```

Upon running the executable, we notice that it requires us to input a password.

```
dynadolod@dystopia$ ./binary
*********************************
  welcome to cracking challenge
*********************************
Enter you password:
```

We then run strings on the file to check for any interesting strings.

```
dynadolos@dystopia$ strings binary
/lib64/ld-linux-x86-64.so.2
__gmon_start__
libc.so.6
exit
__isoc99_scanf
puts
__stack_chk_fail
printf
ptrace
__libc_start_main
GLIBC_2.7
GLIBC_2.4
GLIBC_2.2.5
fff.
secrf
aGVn
YXp5f
c2Ez
ZA==f
cGlu
Zy1w
b25n
dH34%(
dH34%(
VUUU
VUUU
VUUU
<*~7
<=t6
l$ L
t$(L
|$0H
flag: 2cb028fec5abdc8e74994141056858d9
*********************************
  welcome to cracking challenge
samir
Flag: %s
Enter you password:
AAAAAAAAAAAA
eWFfcmFnZWw=
;*3$"
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/>
?456789:;<=
 !"#$%&'()*+,-./0123
```

Upon seeing the string `flag: 2cb028fec5abdc8e74994141056858d9` we tried submitting that as a flag. However, it was rejected. Looking back at the output, we noticed another interesting string: `Flag: %s`. This could potentially be the format string used to print out the actual flag.

Let's take a look at any references to this string in Hopper:

![](../media/images/cscamp_2014/elf1/hexvalues.png)

Based on the disassembly, we can see that some hex values are being passed into base64_decode() before being passed to `printf` as a parameter. The flag is quite possibly the base64 decoded form of those hex values. However, why bother doing that if you can make the program do it for you?

![](../media/images/cscamp_2014/elf1/jump.png)

By looking just above the hex values, we notice that there is a jump that omits the printing of the flag. It appears that the flag will only be printed if base64_decode(???) = 'samir'. Could the unknown value be the user input? Let's find out.

First, let's base64 encode the string `samir`...

```python
In [1]: 'samir'.encode('base64').strip()
Out[1]: 'c2FtaXI='
```

and now let's try it out:

```
dynadolos@dystopia$ ./binary
*********************************
  welcome to cracking challenge
*********************************
Enter you password: c2FtaXI=
Flag: ping-pong you pasamir
```

Looks like we solved it. We tried submitting `ping-pong you pasamir` as the flag but it was rejected. At this point, I was quite sure I had this challenge solved. After clarifying with the organizers, it turns out that all that was required was `ping-pong`.

Since we have this solved, let's confirm if our earlier suspicion was indeed correct.
```python
In [1]: x = 'ping-pong'.encode('base64').strip()
In [2]: ['0x' + i[::-1].encode('hex') for i in map(''.join, zip(*[iter(x)]*4))]
Out[2]: ['0x756c4763', '0x7731795a', '0x6e353262']
```
Yup, it's present in the earlier hex values.

![](../media/images/cscamp_2014/elf1/verify.png)
