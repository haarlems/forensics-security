# Solution

1. Where did the victim download the payload from?

The flag format is `SSS{fullrequesturi}`

``` bash
# we list the full request URIs with tshark
$ tshark -nnr future.pcap -Y 'http' -T fields -e http
.request.full_uri
http://158.94.210.88/jaws
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.x86
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.x86
http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.mips
[..]
# we reconstruct the flag
SSS{http://158.94.210.88/596a96cc7bf9108cd896f33c44aedc8a/db0fa4b8db0333367e9bda3ab68b8042.mips}
```

2. Can you tell which bot is it?

The flag format is `SSS{botname}`

``` bash
# we extract all http files with tshark
$ tshark -nnr future.pcap --export-objects http,./http-out
# we list the files extracted
$ ls -l http-out/
-rw-r--r-- 1 dfir dfir 96408 Jul  6 15:14 'db0fa4b8db0333367e9bda3ab68b8042(1).mips'
-rw-r--r-- 1 dfir dfir 66544 Jul  6 15:14 'db0fa4b8db0333367e9bda3ab68b8042(1).x86'
-rw-r--r-- 1 dfir dfir 96408 Jul  6 15:14  db0fa4b8db0333367e9bda3ab68b8042.mips
-rw-r--r-- 1 dfir dfir 97224 Jul  6 15:14  db0fa4b8db0333367e9bda3ab68b8042.mpsl
-rw-r--r-- 1 dfir dfir 66544 Jul  6 15:14  db0fa4b8db0333367e9bda3ab68b8042.x86
-rw-r--r-- 1 dfir dfir  4816 Jul  6 15:14  jaws
# we compute the hashes
dfir@dfir:/data/sss-evidence/chall-lesson6/future$ sha256sum -b http-out/*
5a5be8301b1b61d5ffe08de1b358574f72fad83a739b9e12ae70e93fa6ba5b14 *http-out/db0fa4b8db0333367e9bda3ab68b8042(1).mips
74ee95ea935954d8320594f45a3ed34d956637f399d927d75f080648800106a0 *http-out/db0fa4b8db0333367e9bda3ab68b8042(1).x86
5a5be8301b1b61d5ffe08de1b358574f72fad83a739b9e12ae70e93fa6ba5b14 *http-out/db0fa4b8db0333367e9bda3ab68b8042.mips
85e6515d887fb0ddf498df540c1a71b10438f722d85b95d613e0cbe37b7c4261 *http-out/db0fa4b8db0333367e9bda3ab68b8042.mpsl
74ee95ea935954d8320594f45a3ed34d956637f399d927d75f080648800106a0 *http-out/db0fa4b8db0333367e9bda3ab68b8042.x86
75eadf63fa491843ff2580532080b3e664b37a7acc44a29fdeda3922bee1b6b8 *http-out/jaws
# we check the hashes in VirusTotal
https://www.virustotal.com/gui/file/5a5be8301b1b61d5ffe08de1b358574f72fad83a739b9e12ae70e93fa6ba5b14
https://www.virustotal.com/gui/file/74ee95ea935954d8320594f45a3ed34d956637f399d927d75f080648800106a0
https://www.virustotal.com/gui/file/85e6515d887fb0ddf498df540c1a71b10438f722d85b95d613e0cbe37b7c4261
https://www.virustotal.com/gui/file/75eadf63fa491843ff2580532080b3e664b37a7acc44a29fdeda3922bee1b6b8
# we see the evaluations are all similar to "Trojan.Linux.Mirai.ATTA"
# OR we drop the payload on windows
# which triggers an alert for "Backdoor:Linux/Mirai.FC!MTB"
# we reconstruct the flag
SSS{mirai}
```
