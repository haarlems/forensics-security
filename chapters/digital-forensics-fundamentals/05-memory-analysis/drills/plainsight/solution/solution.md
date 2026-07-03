# Solution

1. What time did the malicious process id start?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

``` bash
# we run pslist and notice a system.exe which is not a regular system application
$ vol -f evidence.mem windows.pslist
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        File output
[..]
9744    7688    conhost.exe     0x8c0c9b1f3080  3       -       1       False   2026-07-01 17:28:53.000000 UTC  N/A     Disabled
7688    1580    system.exe      0x8c0c9d1cf0c0  20      -       1       True    2026-07-01 17:28:53.000000 UTC  N/A     Disabled
# we reconstruct the flag
SSS{2026-07-01-17:28:53}
```

2. What is its physical memory offset?

The flag format is `SSS{offset}`

``` bash
# we see it in the psscan output with the --physical flag
$ vol -f evidence.mem windows.psscan --physical
PID     PPID    ImageFileName   Offset(P)       Threads Handles SessionId       Wow64   CreateTime      ExitTime        File output
[..]
7688    1580    system.exe      0x3af8e0c0      20      -       1       True    2026-07-01 17:28:53.000000 UTC  N/A     Disabled
# we reconstruct the flag
SSS{0x3af8e0c0}
```

3. What path on disk does it run from?

The flag format is `SSS{\path\to\file}`

``` bash
# we look at the process tree
$ vol -f evidence.mem windows.pstree
PID     PPID    ImageFileName   Offset(V)       Threads Handles SessionId       Wow64   CreateTime      ExitTime      Audit    Cmd     Path
[..]
*** 7688        1580    system.exe      0x8c0c9d1cf0c0  20      -       1       True    2026-07-01 17:28:53.000000 UTCN/A      \Device\HarddiskVolume3\Users\test1\AppData\LocalLow\Microsoft\Internet Explorer\system.exe     -       -
# we reconstruct the flag
SSS{C:\Users\test1\AppData\LocalLow\Microsoft\Internet Explorer\system.exe}
```
