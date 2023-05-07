# level01


## Recognition
We can see when checking the content of `/etc/passwd` file,that the password for flag01 is unshadowed :
```bash
$> cat /etc/passwd
[...]
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
[...]
```

Using the `hashid` command we can identify the type of the hash which is DES in this case.
```bash
kali@kali$ hashid 42hDRfypTqqnw
Analyzing '42hDRfypTqqnw'
[+] DES(Unix) 
[+] Traditional DES 
[+] DEScrypt
```

</br>
</br>


## Exploitation
With the unshadowed line from `/etc/passwd` that we got before, we can try to bruteforce it via a dictionary attack.
```bash
kali@kali$ echo 'flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash' > crypt.txt
kali@kali$ john -w /usr/share/wordlists/rockyou.txt crypt.txt
[...]
0g 0:00:00:00 DONE (2023-05-05 12:45) 0g/s 6440p/s 6440c/s 103040C/s 123456..sss
Session completed.
```

John found the password ! We can now display it using :
```bash
kali@kali$ john --show crypt.txt
flag01:abcdefg:3001:3001::/home/flag/flag01:/bin/bash

1 password hash cracked, 0 left
```

Now, we are able to log-in as flag01 and run the `getflag` command.
```bash
flag01@SnowCrash$ getflag
Check flag.Here is your token : f2av5il02puano7naaf6adaaf
```