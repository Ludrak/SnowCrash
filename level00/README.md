# level00

## Recognition

there is a file called john in /usr/sbin accessible by user level00

</br>
</br>

## Exploitation
the content of the file contains the following content :
```bash
level00@SnowCrash$ cat /usr/sbin/john
cdiiddwpgswtgt
```

We can use a public caesar decrypter to try to decrypt the password.
We used [this](https://www.dcode.fr/caesar-cipher) decoder and found the following result using a ROT15 decryption algorithm.

Flag :
```
nottoohardhere
```

We can then log-in as flag00 and run the `getflag` command giving us the password for level01.
```bash
level00@SnowCrash$ getflag
Check flag.Here is your token : x24ti5gi3x0ol2eh4esiuxias
```