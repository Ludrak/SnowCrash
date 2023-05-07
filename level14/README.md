# level14

## Recognition

When logging on level14, there is nothing really appealing, though we can look at the getflag executable that we used since the beggining to get our flags.

By looking at the disassembled code (analysing the control flow) we can see that there are 2 main contraints:
- 1. Anti dynamic analysis with ptrace (PTRACE_TRACEME)
- 2. getuid check

By using a debuger we can bypass both contraints and get any flag from any user id.

</br>
</br>

## Exploitation

To bypass these, we will first run the program with gdb, then break on the address of the ptrace call. then, we have to change its return value in `eax` to any positive value or 0, then continuing the execution until the `getuid` check.
At this point we only have to set `eax` to the effective uid of the flag user from which we want the flag.

To simplify, we made a simple gdb script along with a bash script to get the flag of any user by exploiting getflag.

```bash
level14@SnowCrash:~$ sh exploit.sh flag14
# break on ptrace address
b *0x804898e
r
# set to eax to bypass ptrace(PTRACE_ME)
set $eax = 0

# break on getuid address
b *0x8048b02
continue

# set eax to the desired user id
set $eax = 3014

# continue the script and get the flag
continue

quit
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
Breakpoint 1 at 0x804898e

Breakpoint 1, 0x0804898e in main ()
Breakpoint 2 at 0x8048b02

Breakpoint 2, 0x08048b02 in main ()
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 2571) exited normally]
```