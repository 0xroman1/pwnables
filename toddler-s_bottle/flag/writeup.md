This challenge just gives us a binary, and tells us that it is a reversing problem. Let't take a look and see what the file is.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ file flag
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked,
for GNU/Linux 2.6.24, BuildID[sha1]=96ec4cc272aeb383bd9ed26c0d4ac0eb5db41b16, not stripped
```

So we know that it is a 64 bit linux executable. Let's run it.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ sudo chmod +x flag
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ ./flag
I will malloc() and strcpy the flag there. take it.
```

The fact that it tells us that we will have to take the flag from the program, and that it doesn't appear to need any input means that we will probably have to use a debugger, such as gdb, to complete this challenge. Let's try that.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ gdb ./flag
```

one wall of text latter...

```
gdb-peda$ disas main
No symbol table is loaded.  Use the "file" command.
gdb-peda$ info functions
All defined functions:
```

That's wierd, it doesn't have a main function, or any function that gdb can tell. This binary has clearly been tampered with. We could try running strings on the program, which should read all of the strings in the program. This might tell us something.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ strings flag
```

Now this will give us a wall of text. However on the last couple of lines, we see something interesting.

```
makBN
su`"]R
UPX!
UPX!
```

We see that the binary contained the string "UPX". UPX (Ultimate Packer for Executables) is an open-source software that will compress executables. Let's see what else we can find from the program related to UPX.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ strings flag | grep UPX
UPX!
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 3.08 Copyright (C) 1996-2011 the UPX Team. All Rights Reserved. $
UPX!
UPX!
```

It says it right there, that this file was packed (compressed) with UPX. So we should be able to decompress it using UPX. If you are on Ubuntu, you can install UPX with the following command.

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ sudo apt-get install UPX
```

And you can uncompress the file like this...

```
guyinatuxedo@tux:/Hackery/pwnablekr/flag$ sudo upx -d flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2013
UPX 3.91        Markus Oberhumer, Laszlo Molnar & John Reiser   Sep 30th 2013

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    887219 <-    335288   37.79%  linux/ElfAMD   flag

Unpacked 1 file.
```

So now that we have successfully unpacked the file, we should be able to read the functions in gdb (as long as they didn't do anything else to them).

```
gdb-peda$ disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   rbp
   0x0000000000401165 <+1>:	mov    rbp,rsp
   0x0000000000401168 <+4>:	sub    rsp,0x10
   0x000000000040116c <+8>:	mov    edi,0x496658
   0x0000000000401171 <+13>:	call   0x402080 <puts>
   0x0000000000401176 <+18>:	mov    edi,0x64
   0x000000000040117b <+23>:	call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:	mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:	mov    rsi,rdx
   0x0000000000401192 <+46>:	mov    rdi,rax
   0x0000000000401195 <+49>:	call   0x400320
   0x000000000040119a <+54>:	mov    eax,0x0
   0x000000000040119f <+59>:	leave  
   0x00000000004011a0 <+60>:	ret    
End of assembler dump.
```

Now we see something interesting at main+32. It is moving something that gdb has commented as the flag into the rdx register (which appears to be the use of a strcopy() that the elf was talking about). In addition right before that at 
maint+23 it calls malloc(), which is also mentioned by the program when we ran it earlier. Let's try and see what the value of that register is after it copies over the value.

```
gdb-peda$ b *main+39
Breakpoint 1 at 0x40118b
gdb-peda$ r
```

One wall of text later...

```
Breakpoint 1, 0x000000000040118b in main ()
gdb-peda$ x/s $rdx
0x496628:	"UPX...? sounds like a delivery service :)"
```

And just like that, we pwned the binary.

Sources: http://taishi8117.github.io/2015/10/26/pwnable-easy1/




