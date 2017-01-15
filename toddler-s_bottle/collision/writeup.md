Let's take a look at the source code...

```
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

When we look at the main function, we see that it reguires atleast one argument (in addition to the program's name). The argument after the program's name must be 20 bytes, and is the only argument that is used or evaluated.
That argument is then passed to the check_password() function, and if it's output is equal to the hex value stored in the hashcode variable (0x21DD09EC) it will use cat to read the flag.
We will have to reverse the check_password() function to see what input we can give it to give us the desired output.

```
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```

Looking at the first line of code, we see that it takes the input "p" of 20 bytes, converts it and stores it as an array of integers.
Proceeding that it proceeds to add all of them up and returns the value. Getting back to the conversion, and int is 4 bytes. The function recieves 20 bytes of data. Since it stores the input in an int array, the array must have 5 items each consisting of 4 bytes of data. So in order to reach the desired input of 0x21DD09EC, we just need to submit a value to the program that when it is broken up into five blocks and added up, will equal 0x21DD09EC.

Starting off, could we just take 0x21DD09EC and divide it by 5? Well 0x21DD09EC equals the decimal value 568134124, so no. What if we subtracted a value from it, then divided by four?

```
0x21DD09EC - 0x01010104 = 551291112
551291112/4 = 137822778
```

So we were able to subtract 0x01010104 from 0x21DD09EC, and then divide the difference by four. This means that we have the four pieces needed to give us the desired input. So let's try it out!

```
col@ubuntu:~$ ./col `python -c 'print 4*"\x3a\x02\x37\x08" + "\x04\x01\x01\x01"'`
daddy! I just managed to create a hash collision :)
```

And just like that, we pwned the binary!


