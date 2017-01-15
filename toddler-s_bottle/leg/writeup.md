Let's take a look at the c code.

```
#include <stdio.h>
#include <fcntl.h>
int key1(){
	asm("mov r3, pc\n");
}
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
int key3(){
	asm("mov r3, lr\n");
}
int main(){
	int key=0;
	printf("Daddy has very strong arm! : ");
	scanf("%d", &key);
	if( (key1()+key2()+key3()) == key ){
		printf("Congratz!\n");
		int fd = open("flag", O_RDONLY);
		char buf[100];
		int r = read(fd, buf, 100);
		write(0, buf, r);
	}
	else{
		printf("I have strong leg :P\n");
	}
	return 0;
}
```

Looking at the main function, we can tell that it scans for input, and compares it to see if it is equivalent to the combined output 
of the key1, key2, and key3 functions. Let's take a closer look at key1.

```
int key1(){
	asm("mov r3, pc\n");
}
```

So all this function does is it executes the assembly code "mov r3, pc". This will move the value of pc into the r3 register (since we are dealing with the r3 register we can assume that we are in an ARM state). The value of pc in an ARM state is the address after the next address to be executed (so the current address plus 8 bytes). We can find this by looking at part of the assembly file they gave us.

```
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
 > 0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
 > 0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
```

Above we can see the assembly code for the key1 function. We can see that the line of code we were looking at is located at +8 (third line from the top). We can also see that the address that will come after the next one is 0x00008ce4. To confirm that this is true, we can subtract 0x00008ce4 from 0x00008cdc and we should get 8.

```
0x00008ce4 - 0x00008cdc = 8
```

And there, since the hex values count the amount of bytes that confirms that the address that is moved into r3 is 0x00008cdc. The thing about the r3 register is that is what holds the value for the return value. So the key1 function will return the hex value 0x00008cdc.

Let's take a look at key2

```
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
```

Looking at this, we can see that it has a lot more code than key1. However we are only really interested in what the function returns which will be the content of the r3 register. So we are only worried about this part.

```
  "mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
```

Just like with key1, we see that it loads the value of pc into r3. However unlike key1, it also adds 0x4 to the value of r3. So we will have to find the value of pc, then add 4 to it. 

```
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
```

Here we can see the address that pc is being pushed to r3, which is  0x00008d04. This means that the value of pc should be 0x00008d08. Now to add 0x4 to that address.

```
0x00008d08 + 0x4 = 0x00008d0c
```

So key2 should return the hex value 0x00008d0c. Now let's look at the key3 function.

```
int key3(){
	asm("mov r3, lr\n");
}
```

This just moves the value of lr into r3. Lr (Link Register) is used to store the return address of a subroutine, which is what key3 is. The return address should be the address immediately following key3 being called, since that is what will be executed after key3. We can look at the assembly code for main to see that.

```
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
   0x00008d84 <+72>:	add	r2, r4, r3
```

We can see that at 0x00008d7c it calls the key3 address using bl (branch link). We can also see that the address immediately following it is 0x00008d80. So key3 should return the hex value 0x00008d80.

So we know what key1, key2, and key3 return. Since we have to match their cumulative value, we should be able to add them up and get the desired input.

```
0x00008ce4 + 0x00008d0c + 0x00008d80 = 108400
```

So let's try our answer...

```
$ ./leg
Daddy has very strong arm! : 108400
Congratz!
My daddy has a lot of ARMv5te muscle!
```

And just like that, we pwned the binary!





