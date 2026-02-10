The Bear Bash DeBugger
==

`bbdb` uses Bash's internal debugging mechanism to provide a kind of debugger which will help you to trace and understand the execution of Bash scripts, even large and multi-files ones.

The most basic invocation is `-f` to trace function entries and exits (with exit code) :
```
$ bbdb -f acme.sh >/dev/null
!/usr/bin/acme.sh:606 | -> _exists()
!/usr/bin/acme.sh:562 |   -> _debug3()
!/usr/bin/acme.sh:606 |   <- 0
!/usr/bin/acme.sh:238 | <- 0
!/usr/bin/acme.sh:8349| -> main()
!/usr/bin/acme.sh:8345|   -> showhelp()
!/usr/bin/acme.sh:7149|     -> _initpath()
!/usr/bin/acme.sh:2919|       -> __initHome()
!/usr/bin/acme.sh:2807|         -> _debug()
!/usr/bin/acme.sh:355 |           -> _log()
!/usr/bin/acme.sh:2807|           <- 0
!/usr/bin/acme.sh:2919|         <- 0
!…me/account.conf:2814|         -> source()
!/usr/bin/acme.sh:2919|         <- 0
...
!/usr/bin/acme.sh:8349|   <- 0
!/usr/bin/acme.sh:8345E status 0
```

If you want to see function args, add `-a` (args are displayed one by line, they are often large strings) :
```
$ bbdb -f -a acme.sh >/dev/null
...
!/usr/bin/acme.sh:2941|       -> _readaccountconf()
!/usr/bin/acme.sh:2941|       -> | [DEFAULT_ACME_SERVER]
!/usr/bin/acme.sh:2527|         -> _read_conf()
!/usr/bin/acme.sh:2527|         -> | [/etc/acme/account.conf]
!/usr/bin/acme.sh:2527|         -> | [DEFAULT_ACME_SERVER]
!/usr/bin/acme.sh:2439|           -> _startswith()
!/usr/bin/acme.sh:2439|           -> | [https://acme-v02.api.letsencrypt.org/directory]
!/usr/bin/acme.sh:2439|           -> | [__ACME_BASE64__START_]
!/usr/bin/acme.sh:2527|           <- 0
...
```

If you know what you're searching for, restrict tracing to functions or statements matching a regex with `-s` :
```
$ bbdb -f -s initHome acme.sh >/dev/null 
!/usr/bin/acme.sh:2919|       -> __initHome()
!/usr/bin/acme.sh:2919|         <- 0
!/usr/bin/acme.sh:8345E status 0
```

You can opt to see all variables assignments with `-w` :
```
$ bbdb -w acme.sh >/dev/null
...
!/usr/bin/acme.sh:2825= DEFAULT_CA_HOME ``
!/usr/bin/acme.sh:2828= LE_TEMP_DIR ``
!…me/account.conf:2 = USER_PATH `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
!…me/account.conf:3 = DEFAULT_ACME_SERVER `https://acme-v02.api.letsencrypt.org/directory`
...
```

You can timestamp your traces with absolute time (`-T`), relative time (`-t`), or both :
```
$ bbdb -f -t -T acme.sh >/dev/null
!11:30:06  0.053 /usr/bin/acme.sh:606 | -> _exists()
!11:30:06  0.056 /usr/bin/acme.sh:562 |   -> _debug3()
!11:30:06  0.057 /usr/bin/acme.sh:606 |   <- 0
!11:30:06  0.058 /usr/bin/acme.sh:238 | <- 0
!11:30:06  0.136 /usr/bin/acme.sh:8349| -> main()
!11:30:06  0.137 /usr/bin/acme.sh:8345|   -> showhelp()
!11:30:06  0.138 /usr/bin/acme.sh:7149|     -> _initpath()
```

And you can suspend execution at a specific point with `-b` (your options are then only to continue or stop the script) :
```
$ bbdb -b /usr/bin/acme.sh:2919 acme.sh >/dev/null
!/usr/bin/acme.sh:2919$ __initHome
!<break>
```

It's possible to combine all of these options if that makes sense for you :
```
$ bbdb -a -f -t -T -w acme.sh >/dev/null
...
!11:34:19  0.105 /usr/bin/acme.sh:455 = LANG `en_US.UTF-8`
!11:34:19  0.116 /usr/bin/acme.sh:601 = _URGLY_PRINTF ``
!11:34:19  0.123 /usr/bin/acme.sh:606 = _ESCAPE_XARGS ``
!11:34:19  0.124 /usr/bin/acme.sh:606 | -> _exists()
!11:34:19  0.125 /usr/bin/acme.sh:606 | -> | [xargs]
!11:34:19  0.126 /usr/bin/acme.sh:549 = cmd ``
!11:34:19  0.129 /usr/bin/acme.sh:562 = ret ``
!11:34:19  0.130 /usr/bin/acme.sh:562 |   -> _debug3()
!11:34:19  0.130 /usr/bin/acme.sh:562 |   -> | [xargs exists=0]
!11:34:19  0.132 /usr/bin/acme.sh:606 |   <- 0
!11:34:19  0.133 /usr/bin/acme.sh:238 | <- 0
!11:34:19  0.156 /usr/bin/acme.sh:608 = _ESCAPE_XARGS ``
!11:34:19  0.179 /usr/bin/acme.sh:964 = __USE_EGREP ``
!11:34:19  0.216 /usr/bin/acme.sh:8349| -> main()
!11:34:19  0.217 /usr/bin/acme.sh:8345|   -> showhelp()
!11:34:19  0.218 /usr/bin/acme.sh:7149|     -> _initpath()
!11:34:19  0.219 /usr/bin/acme.sh:2917= domain ``
!11:34:19  0.220 /usr/bin/acme.sh:2918= _ilength ``
!11:34:19  0.221 /usr/bin/acme.sh:2919|       -> __initHome()
...
```


Options
---

```
$ bbdb -h
bbdb [OPTIONS] [--] program args ...

Option :
  -a            Show function call args
  -b PATH:LINE  Add a breakpoint
  -f            Trace function entry/exit
  -h            This help
  -s RE         Select functions/statements/vars by regex
  -t            Show relative time (precision: ms)
  -T            Show absolute time (precision: s)
  -w            Watch for variable assignments
  -W COLS       Set size for function name column
  -x            Trace all statements
```
