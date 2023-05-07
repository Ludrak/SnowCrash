# level13

## Recognition

There is a program named level13 a the home of the user level13, which is owned by flag13.
```bash
level13@SnowCrash:~$ ls -l
total 8
-rwsr-sr-x 1 flag13 level13 7303 Aug 30  2015 level13
```

When ran, the program tells us that it needs to be ran using UID 4242.
```bash
level13@SnowCrash:~$ ./level13 
UID 2013 started us but we we expect 4242
```

When looking at the .rodata section of the program, we can find a string that looks like a flag, though it is not unfortunately. We can also see the "your token is" sentence, which indicates us that the program can output the valid token. 
```bash
level13@SnowCrash:~$ readelf -x .rodata level13 

Hex dump of section '.rodata':
  0x080486b8 03000000 01000200 30313233 34353600 ........0123456.
  0x080486c8 55494420 25642073 74617274 65642075 UID %d started u
  0x080486d8 73206275 74207765 20776520 65787065 s but we we expe
  0x080486e8 63742025 640a0062 6f655d21 61693046 ct %d..boe]!ai0F
  0x080486f8 42402e3a 7c4c366c 40413f3e 714a7d49 B@.:|L6l@A?>qJ}I
  0x08048708 00796f75 7220746f 6b656e20 69732025 .your token is %
  0x08048718 730a00                              s..
```

When disassembled, we can take a look at the main function, which executes the ft_des function on the token found on the .rodata, this might be the desencryption algorithm used to decrypt the token.
```bash
level13@SnowCrash:~$ objdump -M intel -D level13
[...]
0804858c <main>:
 804858c:       55                      push   ebp
 804858d:       89 e5                   mov    ebp,esp
 804858f:       83 e4 f0                and    esp,0xfffffff0
 8048592:       83 ec 10                sub    esp,0x10
 8048595:       e8 e6 fd ff ff          call   8048380 <getuid@plt>
 804859a:       3d 92 10 00 00          cmp    eax,0x1092         // 4242
 804859f:       74 2a                   je     80485cb <main+0x3f>
 80485a1:       e8 da fd ff ff          call   8048380 <getuid@plt>
 80485a6:       ba c8 86 04 08          mov    edx,0x80486c8      // address of "UID %d started us but we expect %d"
 80485ab:       c7 44 24 08 92 10 00    mov    DWORD PTR [esp+0x8],0x1092
 80485b2:       00 
 80485b3:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 80485b7:       89 14 24                mov    DWORD PTR [esp],edx
 80485ba:       e8 a1 fd ff ff          call   8048360 <printf@plt>
 80485bf:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 80485c6:       e8 d5 fd ff ff          call   80483a0 <exit@plt>
 80485cb:       c7 04 24 ef 86 04 08    mov    DWORD PTR [esp],0x80486ef
 80485d2:       e8 9d fe ff ff          call   8048474 <ft_des>
 80485d7:       ba 09 87 04 08          mov    edx,0x8048709    // address of "your token is %s"
 80485dc:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 80485e0:       89 14 24                mov    DWORD PTR [esp],edx
 80485e3:       e8 78 fd ff ff          call   8048360 <printf@plt>
 80485e8:       c9                      leave  
 80485e9:       c3                      ret 
[...]
```
</br>
</br>

## Exploitation

To exploit this program, we actually need to decrypt the encrypted password in the .rodata section, we could try to reverse the ft_des assembly code or decompile it to C using ghidra for example. however a much simpler solution would be to run the program with a debugger like gdb and edit the register containing the uid value returned by `getuid()`.

Since we have disassembled the program, we can see that the address of the comparaison between `eax` containing the uid and 4242 is `0x804859a`. we can try to break on that address and change the value of `eax` at this time, the program should then give us the decrypted token.

```bash
level13@SnowCrash:~$ ./level13 
UID 2013 started us but we we expect 4242
level13@SnowCrash:~$ gdb level13
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/user/level13/level13...(no debugging symbols found)...done.
(gdb) b *0x804859a
Breakpoint 1 at 0x804859a
(gdb) r
Starting program: /home/user/level13/level13 

Breakpoint 1, 0x0804859a in main ()
(gdb) set $eax = 4242
(gdb) continue
Continuing.
your token is 2A31L79asukciNyi8uppkEuSx
[Inferior 1 (process 5454) exited with code 050]
```

This gives us the password for the user level14.