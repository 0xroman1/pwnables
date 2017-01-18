#Objective:

We all make mistakes, let's move on.
(don't take this too seriously, no fancy hacking skill is required at all)

This task is based on real event
Thanks to dhmonkey

hint : operator priority

ssh mistake@pwnable.kr -p2222 (pw:guest)

#Solution:

When we run the program, it asks us for input, a password, and then says we got the password wrong. Let’s look at the source code to get a better understanding of what’s going on.

```
#include <stdio.h>
#include <fcntl.h>


#define PW_LEN 10
#define XORKEY 1


void xor(char* s, int len){
    int i;
    for(i=0; i<len; i++){
        s[i] ^= XORKEY;
    }
}


int main(int argc, char* argv[]){
    
    int fd;
    if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
        printf("can't open password %d\n", fd);
        return 0;
    }


    printf("do not bruteforce...\n");
    sleep(time(0)%20);


    char pw_buf[PW_LEN+1];
    int len;
    if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
        printf("read error\n");
        close(fd);
        return 0;        
    }


    char pw_buf2[PW_LEN+1];
    printf("input password : ");
    scanf("%10s", pw_buf2);


    // xor your input
    xor(pw_buf2, 10);


    if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
        printf("Password OK\n");
        system("/bin/cat flag\n");
    }
    else{
        printf("Wrong Password\n");
    }


    close(fd);
    return 0;
}
```

Looking at this code, the first important thing it does is it scans for input and stores it under “pw_buf” which that is the initial input we gave it. Proceeding that it will again scan for input after printing “input password” and store it as “pw_buf2”. Proceeding that it will run “pw_buf2” through a function “xor” which will xor each character of that input with “XORKEY” defined earlier in the code, which is “1”. If you want to learn more about how XOR works checkout this link.

https://en.wikipedia.org/wiki/XOR_cipher

Proceeding after it checks to see if the values of “pw_buf” and “pw_buf2” are equal using strncmp (the PW_LEN is only there to specify that it should be 10 characters). Strncmp will output a 0 if the output matches, which in c is considered false so that is why the “!” is there. So essentially we will need to input one 10 character value, and another 10 character value that when xored with 1 will give us the original value. For this we can just put “0000000000” as the first value, and “1111111111” as the second since “1”^”1” = 0.

```
./mistake
do not bruteforce...
0000000000
input password : 1111111111
Password OK
Mommy, the operator priority always confuses me :(
```







