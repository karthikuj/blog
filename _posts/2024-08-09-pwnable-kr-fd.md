---
layout: post
title:  "Pwnable.kr: fd Writeup"
date:   2024-08-09 10:03:24 +0530
categories: binary-exploitation binex-labs
---

### Challenge URL

[Pwnable.kr](https://pwnable.kr/play.php)


### Challenge

We are given a binary (`fd`), a C file (`fd.c`), and a flag (`flag`). We do not have the permission to view the flag but we can somehow manipulate the input to the binary to get the flag.

Let's copy the necessary files from pwnable.kr's server to our local machine.
{% highlight shell %}
ssh -p 2222 fd@pwnable.kr
# Enter password 'guest' when prompted
ls
{% endhighlight %}

{% highlight shell %}
scp -P 2222 fd@pwnable.kr:/home/fd/fd.c .
# Enter password 'guest' when prompted
vi fd.c
{% endhighlight %}


### Solution

Now we have the following C code:
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
{% endhighlight %}

According to the code, first it will check if the argument count (`argc`) is at least 2, if it is not then it will print **_"pass argv[1] a number"_** and exit. We know that whenever we run an executable the first argument is the path to the binary itself, so we need to pass 1 more argument and based on the exit prompt, it needs to be a number.

Now when we pass a number as an argument, the program will put that in a function `atoi()`. According to the docs:
> The atoi() function converts the initial portion of the string pointed to by nptr to int.

It will then subtract `0x1234` or `4660` (in decimal) from that integer, the result is then assigned to an integer variable called `fd`. The fd variable is later being passed to `read()` function as a file descriptor and the `read()` function is storing the input to a 32 bytes sized buffer. A file descriptor is just a number we get when we open a connection to a file and in Linux almost everything is a file, we also know that in Linux the file descriptor 0 is for stdin, 1 is for stdout and 2 is for stderr.

This means that if we pass the number `4660` as the argument to the binary, the value of the fd variable will be 0:
{% highlight python %}
fd = atoi(4660) - 0x1234
fd = 4660 - 0x1234
# Remember that the value of 0x1234 is 4660 in decimal
fd = 0
{% endhighlight %}
So, now we can pass the input to the program through stdin. Okay, now the program checks if the value in the buffer is **"LETMEWIN\n"**, if it is then it will print out the flag, this looks easy enough, we just need to enter that as the input to the program.

Here's a POC to make it faster:
{% highlight shell %}
ssh -p 2222 fd@pwnable.kr "echo -ne \"LETMEWIN\n\" | ./fd 4660"
# Enter password 'guest' when prompted
{% endhighlight %}

This will get us our flag.
Happy hacking :D