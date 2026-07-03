# Solution

``` bash
# first we check the image for info
$ vol -f ftk.mem windows.info
Variable        Value
Kernel Base     0xf803c1000000
DTB     0x1ae000
Symbols file:~/vol3-venv/lib/python3.9/site-packages/volatility3/symbols/windows/ntkrnlmp.pdb/11D7FE79CC245612055503EE86BB0E3E-1.json.xz
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 FileLayer
KdVersionBlock  0xf803c1e0a9a0
Major/Minor     15.26100
MachineType     34404
KeNumberProcessors      2
SystemTime      2026-06-18 08:10:50+00:00
NtSystemRoot    C:\WINDOWS
NtProductType   NtProductWinNt
NtMajorVersion  10
NtMinorVersion  0
PE MajorOperatingSystemVersion  10
PE MinorOperatingSystemVersion  0
PE Machine      34404
PE TimeDateStamp        Mon Dec  3 21:59:54 2046
# we list the sessions to see per process usernames
# we find a systemid user, hinted in the description
# we find a notepad spawned by powershell
$ vol -f ftk.mem windows.sessions
Session ID      Session Type    Process ID      Process User Name       Create Time
[..]
1       -       10084   powershell.exe  test/systemid   2026-06-18 08:09:41.000000 UTC
1       -       10092   conhost.exe     test/systemid   2026-06-18 08:09:41.000000 UTC
1       -       3760    notepad.exe     test/systemid   2026-06-18 08:09:58.000000 UTC
# we list the processes
$ vol -f ftk.mem windows.pslist
[..]
10084   10012   powershell.exe  0xe2819f2fb080  12      -       1       False   2026-06-18 08:09:41.000000 UTC  N/A     Disabled
3760    10084   notepad.exe     0xe2819b9e80c0  1       -       1       False   2026-06-18 08:09:58.000000 UTC  N/A     Disabled
# we dump the memory of the notepad process
$ vol -f ftk.mem windows.memmap --pid 3760 --dump
# we grep for the flag with strings for UTF-16LE strings, common in Windows
$ strings -e l pid.3760.dmp | grep -i flag
The flag is RkZGe3N2WTNfZTNwYklyZUxfanZNNGVxZWx9
# we base64 rot13 decode it to get the flag
$ echo "RkZGe3N2WTNfZTNwYklyZUxfanZNNGVxZWx9" | base64 -d | tr 'A-Za-z' 'N-ZA-Mn-za-m'
SSS{fiL3_r3coVerY_wiZ4rdry}
```
