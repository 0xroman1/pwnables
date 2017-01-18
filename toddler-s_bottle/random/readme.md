#Objective:
This is a 1 pt challenge from pwnable.kr

Daddy, teach me how to use random value in programming!

ssh random@pwnable.kr -p2222 (pw:guest)

#Solution:

To start off let’s ssh into the server and see what we have. When it asks for a password type in “guest”.

```
ssh random@pwnable.kr -p2222
```

When we log in, we can see using the “ls” command that there are three files. “flag”, “random”, and “random.c”. Let’s try to run the “random” application and see what happens (we can confirm it is an executable by using the “file” command on it).

```
“./random
What do you do?
Wrong, maybe you should try 2^32 cases.
```

Ok from this we can inference from this and the challenge description that it is probably checking our input against some random value. Let’s take a look at the c code for it.

```
#include <stdio.h>


int main(){
    unsigned int random;
    random = rand();    // random value!


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

Ok looking at here, it first generates a random value “random” using the c rand() function. Next it takes our input, and stores it as an int named “key”. It then checks to see if “key” xored with “random” is equal to the hex value “0xdeadbeef” (for more information about how xor works http://bfy.tw/8lek). There is a red flag with the rand() function. For starters that function is not cryptographically secure, so it is only good for things that a random number really doesn’t matter (like in a game). The major issue with te rand() function is that it requires a seed to generate values, which it does not currently have. So it will just output the same number over and over again. So if we can just find out what that number is, we can just xor that number with the decimal value of “0xdeadbeef” and we will have the input needed to win the challenge. To do this I just copy and pasted the code into a new C project in codeblocks and ran it, accept I added a line of code that will print out the value of “random”. 

```
int main(){
    unsigned int random;
    random = rand();    // random value!
    printf("%d\n", random);
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

Just like that we get the value for “random”, which is 1804289383. Now we just have to xor it with the hex value 0xdeadbeef and we should have our answer. You can use an online xor calculator, python, or anything you want.

```
0xdeadbeef = 3735928559
3735928559 ^ 1804289383 = 3039230856
```

Now that we have the value that we need to meet the requirements of the if then statement, we should just be able to get the flag.

```
./random
3039230856
Good!
Mommy, I thought libc random is unpredictable…
```


