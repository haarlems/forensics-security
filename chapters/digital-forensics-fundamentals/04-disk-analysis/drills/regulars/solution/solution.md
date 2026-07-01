# Solution

``` bash
# we check the systemd journal
$ journalctl --directory \[root\]/var/log/journal/ --utc _UID=0 -o verbose
[..]
Mon 2026-06-29 15:42:58.765736 UTC [s=9fbac86c19a74229aca807b62895db81;i=24a92;b=8286180fa28b44ce9720>
    PRIORITY=6
    _UID=0
    _GID=0
[..]
    SYSLOG_IDENTIFIER=useradd
    _COMM=useradd
    SYSLOG_PID=3616
    SYSLOG_TIMESTAMP=Jun 29 18:42:58
    MESSAGE=new user: name=avahi-d, UID=0, GID=0, home=/home/avahi-d, shell=/bin/sh, from=/dev/pts/2
    _PID=3616
    _EXE=/usr/sbin/useradd
    _CMDLINE=useradd -o -u 0 -g 0 -m avahi-d
    _SOURCE_REALTIME_TIMESTAMP=1782747778765736
# or we check the auth.log
$ cat \[root\]/var/log/auth.log
[..]
Jun 29 12:03:58 ritabeakerlab sudo: ritabeakerlab : TTY=pts/0 ; PWD=/home/ritabeakerlab ; USER=root ; COMMAND=/usr/sbin/useradd -o -u 0 -g 0 -m avahi-d
# we reconstruct the flag
SSS{/usr/sbin/useradd}
```

``` bash
# we check the systemd journal
$ journalctl --directory \[root\]/var/log/journal/ --utc _UID=0 -o verbose | grep _CMDLINE
# we reconstruct the flag
SSS{useradd -o -u 0 -g 0 -m avahi-d}
```

``` bash
# we check the bashrc for the backdoor user and find a weird addition
$ cat \[root\]/home/avahi-d/.bashrc
[..]
# avahi-daemon config
(/var/log/avahi/.d/run.sh >/dev/null 2>&1 &)
SSS{/var/log/avahi/.d/run.sh}
```

``` bash
# we see timestamp for the backdoor user .bashrc in the bodyfile
Line	Tag	Timestamp	macb	Meta	File Name	File Size
2409141	Unchecked	2026-06-29 16:14:58	m.cb	2248618	/home/avahi-d/.bashrc	3837
# we reconstruct the flag
SSS{2026-06-29-16:14:58}
```

``` bash
# we check who logged in
$ last -if \[root\]/var/log/wtmp
# the only user besides our beachhead user is avahi-d
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:15 - 16:19  (00:03)
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:10 - 16:15  (00:04)
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:06 - 16:10  (00:03)
# we check what shell they use
# we notice they also use the root uid and gid 0
$ cat /etc/passwd
[..]
sys-gnd:x:1001:1001:,,,:/home/sys-gnd:/bin/bash
xfcenv:x:1002:1002:,,,:/home/xfcenv:/bin/bash
admin:x:1003:1003:,,,:/home/admin:/bin/bash
avahi-d:x:0:0::/home/avahi-d:/bin/sh
# we reconstruct the flag
SSS{/home/avahi-d:/bin/sh}
```

``` bash
# we check who logged in and see the timestamps
$ last -if \[root\]/var/log/wtmp
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:15 - 16:19  (00:03)
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:10 - 16:15  (00:04)
avahi-d  pts/0        fe80::2eb0:ae7b: Mon Jun 29 16:06 - 16:10  (00:03)
# we reconstruct the flag
SSS{2026-06-29-16:06}
```

``` bash
# we check the btmp log
$ last -if \[root\]/var/log/btmp
hp       ssh:notty    fe80::2eb0:ae7b: Mon Jun 29 14:32    gone - no logout
hp       ssh:notty    fe80::2eb0:ae7b: Mon Jun 29 14:32 - 14:32  (00:00)
# we reconstruct the flag
SSS{hp}
```
