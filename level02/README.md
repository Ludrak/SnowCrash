# level02

## Recognition

We can find a file named level02.pcap in the home folder.
```bash
level02@SnowCrash$ ls -l
total 12
----r--r-- 1 flag02 level02 8302 Aug 30  2015 level02.pcap
```

Pcap files are snapshots of a network sniff, it contains the data exchanged between multiples machines on a specific network.

</br>
</br>

## Exploitation

We can open and inspect its content using Wireshark.

While inspecting with Wireshark, we found out that this snapshot was unencrypted and was a communication between only two machines at 59.233.235.218 and 59.233.235.223.

We can then follow the TCP Stream to see what data was exchanged.

TCP Stream level02.pcap:
```
..%..%..&..... ..#..'..$..&..... ..#..'..$.. .....#.....'........... .38400,38400....#.SodaCan:0....'..DISPLAY.SodaCan:0......xterm.........."........!........"..".....b........b....	B.
..............................1.......!.."......"......!..........."........"..".............	..
.....................
Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10)

..wwwbugs login: l.le.ev.ve.el.lX.X
..
Password: ft_wandr...NDRel.L0L
.
..
Login incorrect
wwwbugs login: 
```

As we can see, this is a snapshot from a login connection. Since the connection is unencrypted we can see the password content entered by the user.

Looking further, the password also contains the ASCII character `7f` which is the `DEL` character, from that we can deduce that the user mistyped his password. We can easily recompose it like that :

```
ft_waNDReL0L
```

Now, we are able to log in as flag02 and run the `getflag` command.
```bash
flag02@SnowCrash:~$ getflag
Check flag.Here is your token : kooda2puivaav1idi4f57q8iq
```