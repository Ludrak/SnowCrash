# level09

## Recognition

There is an executable which takes in a string as first argument and shifts all its charaters by their index in the string.
```bash
level09@SnowCrash:~$ ./level09
You need to provied only one arg.
level09@SnowCrash:~$ ./level09 'hello world'
hfnos%}vzun
level09@SnowCrash:~$ ./level09 aaaaaaaaaaaaa
abcdefghijklm
```

There is also a file named 'token' which contains a strange string that might be interesting.
```bash
level09@SnowCrash:~$ cat token
f4kmm6p|=�p�n��DB�Du{��
```

</br>
</br>

## Exploitation

As we have seen before the token file might contain an encrypted password in which each character is shifted by its index.

We can then make a script to reverse the operation.
We used [this](./resources/exploit.c) script to reverse the password.

```bash
level09@SnowCrash:~$ chmod +rwx .
level09@SnowCrash:~$ gcc exploit.c -o exploit -std=c99
level09@SnowCrash:~$ ./exploit $(cat token)
f3iji1ju5yuevaus41q1afiuq
```

We can then use this token to log as flag09 and run `getflag`.
```
flag09@SnowCrash:~$ getflag
Check flag.Here is your token : s5cAJpM8ev6XHw998pRWG728z
```