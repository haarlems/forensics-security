# Memory analysis

A wealth of evidence resides in the random access memory (RAM) of the analyzed system, which can be used to correlate with other evidence, such as network or disk activity.<br />
When the system is shutdown, the memory data is lost.

The main focus of memory analysis is finding malicious processes or executables to extract and examine further.

## Prerequisites

- a unix forensic vm with
  - volatility3
  - nice to have but not mandatory
    - volatility2

## Memory artifacts

- network connections
  - IP and port source and destination, state of the connection, protocols
- running processes
  - each process has its own memory space and is tracked via process IDs (PID)
- loaded DLLs (Dynamic Link Libraries)
- loaded drivers
- handles
  - used by processes to interact with system resources (files, registry keys, etc.)
- command history
- open registry keys
- credentials
  - session tokens, plaintext app passwords, encryption keys

### Identify rogue processes

- process name (disguised to mimic legitimate processes)
- process path (executed from illegitimate source)
- suspicious ppid-pid (for example notepad.exe spawning cmd.exe)

### Identify library loading and handles

- examine DLLs associated with the rogue process
- look for unfamiliar DLL names or paths (ex: Temp or AppData are non standard DLL locations)
- look for DLLs with timestamps inconsistent with OS installation date

### Identify network artifacts

- connections to suspicious external hosts: Command and Control (C2) beacons
- extract PID and process name initiating the connection
- analyze PPID to confirm or deny legitimacy
- check associated objects: DLLs, handles, reg keys

### Identify code injection

- DLL injection, reflection, process hollowing: `windows.memmap` volatility plugin to identify DLLs in memory but not on disk
- remote thread creation: `windows.threads`
- look for unmapped sections of memory

### Identify persistence

- signs of rootkits
- signs of malicious drivers
- examine autostart keys

### Dump suspicious files from process memory

- dump files from process memory space to investigate further

## Volatility

Volatility and Volatility3 are open source memory forensics frameworks.
They can run on any common OS supporting Python.

The difference between them lies in capability and usage:

- `Volatility` or `Volatility2` is the original version developed in python2, with multiple plugins and functionality
- `Volatility3` is the newer version developed in python3, with most of the functionality of 2 ported, but not all, yet

We will use Volatility3, and resort to Volatility2 only in case of missing functionality.

### General information

``` bash
# we scan for windows os and image metadata
$ python3 vol.py -f win.dmp windows.info 

# we scan for unix kernel version details
$ python3 vol.py -f unix.dmp banners 
```

To analyze a Linux or macOS RAM capture with Volatility3 you must to use appropriate kernel debugging information.

We find a collection of [symbol](https://github.com/Abyss-W4tcher/volatility3-symbols) tables.

If you cannot find an appropriate symbol table for your kernel version, you can also [create one](https://volatility3.readthedocs.io/en/latest/symbol-tables.html#mac-or-linux-symbol-tables) manually.

``` bash
# download the matching debug kernel vmlinux with dwarf
# example ubuntu
$ sudo apt install linux-image-6.17.0-35-generic-dbgsym
# convert with dwarf2json
$ git clone https://github.com/volatilityfoundation/dwarf2json.git
$ cd dwarf2json && go build
$ ./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-6.17.0-35-generic > 6.17.0-35-generic.json
# xz the json and move to volatility symbols directory
$ xz 6.17.0-35-generic.json
$ mv 6.17.0-35-generic.json.xz ~/venv/lib/python3.9/site-packages/volatility3/symbols
```

### Linux

``` bash
# list all processes linked list
python3 vol.py -f unix.dmp linux.pslist
OFFSET (V)      PID     TID     PPID    COMM    UID     GID     EUID    EGID    CREATION TIME   File output
0x898d00a2a9c0  1       1       0       systemd 0       0       0       0       -       Disabled
[..]
# scan memory for process structures
python3 vol.py -f unix.dmp linux.psscan
# show process tree
python3 vol.py -f unix.dmp linux.pstree
OFFSET (V)      PID     TID     PPID    COMM
0x898d00a2a9c0  1       1       0       systemd
* 0x898d061d0000        370     370     1       systemd-journal
* 0x898d23ce5380        861     861     1       avahi-daemon
** 0x898d25f28000       919     919     861     avahi-daemon
[..]
# show processes with command-line args
python3 vol.py -f unix.dmp linux.psaux
PID     PPID    COMM    ARGS
1       0       systemd /sbin/init splash
2       0       kthreadd        [kthreadd]
3       2       pool_workqueue_ [pool_workqueue_]
[..]
# list open files per process
python3 vol.py -f unix.dmp linux.lsof
# inspect network connections
python3 vol.py -f unix.dmp linux.netscan
# inspect netfilter hooks
python3 vol.py -f unix.dmp linux.netfilter
# show command history
python3 vol.py -f unix.dmp linux.bash
# scan for suspicious memory regions
python3 vol.py -f unix.dmp linux.malfind
# scan with yara rules
python3 vol.py -f unix.dmp linux.vmayarascan.VmaYaraScan --yara-file=rule.yar
```

### Windows

``` bash
# list all processes linked list
python3 vol.py -f win.dmp windows.pslist
# scan for all EPROCESS structures
python3 vol.py -f win.dmp windows.psscan
# show process tree
python3 vol.py -f win.dmp windows.pstree
# show cross view of process listings
python3 vol.py -f win.dmp windows.psxview
# list dlls loaded by PID
python3 vol.py -f win.dmp windows.dlllist --pid 1337
# extract all memory mapped regions
python3 vol.py -f win.dmp windows.memmap --pid 1337 --dump
# list handles opened by PID
python3 vol.py -f win.dmp windows.handles --pid 1337
# scan for network objects
python3 vol.py -f win.dmp windows.netscan
# show command history
python3 vol.py -f win.dmp windows.cmdline
# scan for rogue activity
python3 vol.py -f win.dmp windows.malfind
# list registry hives mapped in memory, based on OS structures
python3 vol.py -f win.dmp windows.registry.hivelist
# scan raw memory for registry hives
# based on hive signatures
# can find hives not loaded in OS structures, or corrupted hives
python3 vol.py -f win.dmp windows.registry.hivescan 
# extract files from process memory
python3 vol.py -f memory.dmp windows.dumpfiles --pid 1337 -o ~/outputpid1337/ 
# scan with YARA rules
python3 vol.py -f memory.dmp windows.yarascan --yara-file=rule.yar
```

## Strings

Unix utility used to extract strings from memory.

`strings memory.dmp | grep [pattern|regex]` # default for ASCII strings, grep a specific pattern or regular expression

`strings memory.dmp -e l` # for windows UTF-16LE strings

- -e encoding
- l for Windows Unicode UTF-16LE

See `man strings` for other encoding options.

## Summary

- we use `volatility` to analyze the memory dumps
- unix memory dumps require a symbols table for `volatility` to read them
- we look for suspicious processes, running from unexpected locations
- we check for network connections
- we look for the any commands from the shell history
- we dump the files and investigate further

## Drills

### untitled

During a routine audit, a new local account was found on this workstation with no justification for its creation. The system was flagged for unusual activity and a memory capture tas taken. Find the flag.

### plainsight

We collected this memory dump, see if you can make sense of it.

## Further reading

[+] Digital Forensics and Incident Response, chapter 10. Analyzing System Memory, by Gerard Johansen<br />
[+] [SANS Memory Forensics Cheatsheet](https://www.sans.org/posters/memory-forensics)<br />
[+] [Volatility3](https://github.com/volatilityfoundation/volatility3)<br />
[+] [Volatility](https://github.com/volatilityfoundation/volatility)<br />
[+] [Volatility3 Symbols](https://github.com/Abyss-W4tcher/volatility3-symbols)<br />
[+] [Creating New Symbol Tables](https://volatility3.readthedocs.io/en/latest/symbol-tables.html#mac-or-linux-symbol-tables)<br />
