# Evidence acquisition & integrity

Digital forensics is concerned with "analyzing digital evidence in a manner that is legally acceptable in any legal proceedings", thus [evidence integrity](https://www.isaca.org/resources/white-papers/overview-of-digital-forensics) is key.<br />

Before touching any system:

- confirm you have written permission to collect data from it
- define the scope: which systems, what evidence, what time range
- assign an evidence custodian to maintain the chain of custody
- prepare the forensic workstation:
  - write blocker, imaging tools, and hashing tools ready
  - storage media wiped
- make note of every step taken

After an incident has been identified and scoped, next comes the evidence collection phase.<br />
Collected evidence must:

- be relevant and of consequence to proving or disproving a hypothesis
- have its integrity preserved
- be thoroughly documented

Choosing a collection tool depends on:

- is the system live or offline?
- what is the target OS?

## Prerequisites

- a unix forensic vm with
  - TSK
  - UAC
  - tcpdump
  - wireshark
  - LiME or AVML
- a windows forensic vm with
  - EZ Tools
  - KAPE
  - FTK Imager (community edition)

## Preserving integrity

### Write blockers

The forensic workstation doing the acquisition must not write anything back to the source disk.<br />
Even a simple OS action like updating a file's last-accessed timestamp is enough to alter evidence and call its integrity into question.

A **write-blocker** sits between the source device and the forensic workstation, intercepting any write commands and preventing them from reaching the disk while allowing read commands through normally.<br />
Write-blockers come in two forms:

- **Hardware**: a dedicated physical device that connects between the source disk and the forensic workstation
- **Software**: a driver or OS-level tool that blocks writes at the OS level

### Hash verification

It is recommended that a unique hash value per each file collected is calculated.<br/>
A hash is a numeric value of a fixed length that uniquely identifies data, computed based on a chosen algorithm.<br />
Commonly used algorithms:

- sha256
- sha1
- md5

#### Usage examples

``` bash
sha256sum <file>
md5sum <file>
```

``` powershell
> certutil.exe -hashfile .\indicators.csv
SHA1 hash of .\indicators.csv:
c3c60d608ba2ce8dcf6303b84d42f8e8a5cde766
> certutil.exe -hashfile .\indicators.csv SHA256
SHA256 hash of .\indicators.csv:
35fa7037c4218fbf61462f80f250c971bfb318ca1ae6a229f1f93ff6810adffc
> certutil.exe -hashfile .\indicators.csv MD5
MD5 hash of .\indicators.csv:
c9f752858cc253b2a48f712c0f74f3de

> Get-FileHash .\indicators.csv | Format-List
Algorithm : SHA256
Hash      : 35FA7037C4218FBF61462F80F250C971BFB318CA1AE6A229F1F93FF6810ADFFC
Path      : [..]\indicators.csv
> Get-FileHash .\indicators.csv -Algorithm SHA1 | Format-List
Algorithm : SHA1
Hash      : C3C60D608BA2CE8DCF6303B84D42F8E8A5CDE766
Path      : [..]\indicators.csv

```

## Evidence volatility

Evidence collection should be prioritized based on evidence volatility.<br />
According to the IETF [RFC 3227 - Guidelines for Evidence Collection and Archiving](https://www.ietf.org/rfc/rfc3227.txt), when collecting evidence you should proceed from the volatile to the less volatile.<br />
The order of volatility for a typical system:

- registers, cache
- routing table, arp cache, process table, kernel statistics, memory
- temporary file systems
- disk
- remote logging and monitoring data that is relevant to the system in question
- physical configuration, network topology
- archival media

For example, if network logs roll over a 24h period, they should be acquired at the debut of the investigation, and then assess if they are required.

## Evidence acquisition - Host-based

From the hosts we usually collect disk and memory artifacts.<br />
Network captures can also be performed on a single host, but they are usually performed at the edge of a network perimeter.

Full disk images and memory captures are large and must be stored on dedicated forensic storage, not on the same device being investigated, but on an external drive or NAS that has been wiped before.

Never save memory dumps directly on the compromised disk, to avoid overwriting deleted files or artifacts.

Disk artifacts may include:

- **information about file system metadata**
  - linux: inode table, ext4 journal
  - windows: $MFT file, $LogFile
- **evidence of execution**
  - linux: syslog, systemd journal logs, `.bash_history`, `.zsh_history`
  - windows: Prefetch files, SRUM
- **evidence of past file presence**
  - linux: ext4 journal
  - windows: USN Journal, AmCache, Windows Search Index
- **evidence of network communication**
  - linux: `/var/log/ufw.log`, `/var/log/auth.log`
  - windows: BITS Database
- **evidence of access**
  - linux: `.bash_history`, `/var/log/auth.log`,
  - windows: shellbags
- **browser artifacts**
  - cross platform: credentials, cookies, browsing history
  - stored in profile directories for Firefox, Chrome, etc

Memory artifacts may include:

- process information
- network information
- loaded libraries and drivers
- handles
- command history
- credentials, etc

### Disk imaging formats

A forensic disk image is a bit-for-bit copy of a storage device, including unallocated space, deleted files, and file system metadata.<br />
The format affects tool compatibility, storage, and what metadata is preserved.

Common formats:

- **[raw (dd)](https://forensics.wiki/raw_image_format/)**: uncompressed copy with no metadata
  - readable by almost every forensic tool
  - no built-in hash verification or case metadata
  - file size equals the source disk size
- **[E01](https://blog.ecapuano.com/p/mounting-e01-forensic-images-in-linux)**: proprietary format, commonly used
  - supports compression, built-in hash verification, and embedded case metadata
  - splits into manageable segments
  - supported by Autopsy, FTK, and other tools
- **[AFF (Advanced Forensic Format)](https://forensics.wiki/aff/)**: open-source alternative to E01 with similar metadata and compression capabilities
  - less common today

In practice, `E01` is used for most investigations for the integrity verification, compression, and metadata.<br />
`Raw (dd)` is used when interoperability is the priority.

### dd

`dd` is a command-line utility that copies, converts and formats data.
`dc3dd` is a fork of `dd` that does hash verification and logging as well.
We will use it to take a disk image on unix.

``` bash
# first we create a 100MB file to use as a practice disk
$ dd if=/dev/zero of=/data/sss-evidence/practice.img bs=1M count=100
[..]
104857600 bytes (105 MB, 100 MiB) copied, 0.865699 s, 121 MB/s

# we create a filesystem and put some data on it
$ mkfs.ext4 /data/sss-evidence/practice.img
mke2fs 1.46.2 (28-Feb-2021)
[..]
Writing superblocks and filesystem accounting information: done
$ sudo mkdir -p /mnt/practice
$ sudo mount -o loop /data/sss-evidence/practice.img /mnt/practice
$ echo "evidence file content" | sudo tee /mnt/practice/noticeme.txt
evidence file content
$ sudo umount /mnt/practice

# we attach it as a read only device to prevent writes to the source
$ sudo losetup --read-only --find --show /data/sss-evidence/practice.img
/dev/loop0
# we check it's read only
$ sudo blockdev --getro /dev/loop0
1
# we double check it's read only
$ sudo dd if=/dev/zero of=/dev/loop0 bs=512 count=1
dd: error writing '/dev/loop0': Operation not permitted
[..]

# we take a raw image with dc3dd
$ sudo dc3dd if=/dev/loop0 of=/data/sss-evidence/practice.raw hash=sha256 log=/data/sss-evidence/practice.log hlog=/data/sss-evidence/practice.hashes
dc3dd 7.2.646 started at 2026-06-22 09:36:21 +0000
compiled options:
command line: dc3dd if=/dev/loop0 of=/data/sss-evidence/practice.raw hash=sha256 log=/data/sss-evidence/practice.log hlog=/data/sss-evidence/practice.hashes
device size: 204800 sectors (probed),      104,857,600 bytes
sector size: 512 bytes (probed)
   104857600 bytes ( 100 M ) copied ( 100% ),    1 s, 75 M/s
input results for device `/dev/loop0':
   204800 sectors in
   0 bad sectors replaced by zeros
   51aa6ad4ee06375a71c32d6f9cd93ce352859f1c9698c01e0281bb790de8da0a (sha256)
output results for file `/data/sss-evidence/practice.raw':
   204800 sectors out
dc3dd completed at 2026-06-22 09:36:22 +0000
# we verify the hashes
$ sha256sum practice.raw
51aa6ad4ee06375a71c32d6f9cd93ce352859f1c9698c01e0281bb790de8da0a  practice.raw
$ sha256sum practice.img
51aa6ad4ee06375a71c32d6f9cd93ce352859f1c9698c01e0281bb790de8da0a  practice.img
```

### [UAC](https://github.com/tclahr/uac) - Unix-like Artifacts Collector

Live collection tool for Unix-like operating systems.<br />
UAC comes with two basic predefined profiles:

- `ir_triage` captures important system artifacts needed for typical investigations
- `full` everything in ir_triage plus browser and other user application artifacts

Usage:

``` bash
# we collect all artifacts based on the ir_triage profile and save the outputs to /tmp
sss@sss:~/uac-3.3.0$ sudo ./uac -p ir_triage /tmp
--------------------------------------------------------------------------------
  ___     ___
 | \ |   |   |
 | / |   |   |
 |   |   |   |
 |_  |___|   |
   |_________|

 Unix-like Artifacts Collector 3.3.0
--------------------------------------------------------------------------------
Operating System    : linux
System Architecture : x86_64
CPU                 : AMD Ryzen [..] with Radeon Graphics @ 1896 Mhz
Memory Size         : 3960 MB
Hostname            : sss
Time Zone           : Europe/Bucharest
Mount Point         : /
Running as          : root
Temp Directory      : /tmp/uac-data.tmp
--------------------------------------------------------------------------------
Artifacts collection started...
[001/219] 2026-06-22 13:06:36 +0300 live_response/process/ps.yaml
[002/219] 2026-06-22 13:06:37 +0300 live_response/process/lsof.yaml
[003/219] 2026-06-22 13:06:38 +0300 live_response/process/top.yaml
[004/219] 2026-06-22 13:06:38 +0300 live_response/process/procfs_information.yaml
[..]
# we check the hashes
$ cat /tmp/uac-sss-linux-20260622130635.log
Created by UAC (Unix-like Artifacts Collector) 3.3.0
[..]
[System Information]
Operating System: linux
System Architecture: x86_64
Hostname: sss
[Acquisition Information]
Mount Point: /
Acquisition Started: Mon Jun 22 13:06:36 2026 +0300
Acquisition Finished: Mon Jun 22 13:15:43 2026 +0300
[Output Information]
File: uac-sss-linux-20260622130635.tar.gz
Format: tar
[Computed Hashes]
MD5 checksum: c1ba1586c0d7eb5d57cf0e557454751f
SHA1 checksum: b476fec2c450c08fb3f6bc7cd387065a723329f7
# we verify the hashes
$ sha1sum /tmp/uac-sss-linux-20260622130635.tar.gz
b476fec2c450c08fb3f6bc7cd387065a723329f7  /tmp/uac-sss-linux-20260622130635.tar.gz
```

![uac](../media/uac.png)

### [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)

Live collection tool to capture disk and memory on Windows.
It has a free version and multiple capabilities. <br />
Options:

- create disk image
- capture memory

![ftk](../media/ftk.png) <br />

### [EZ Tools](https://ericzimmerman.github.io/)

- KAPE - Kroll Artifact Parser and Extractor
  - can be used to collect the most critical artifacts prior to the start of the imaging process
  - while the imaging completes, the data generated by KAPE can be reviewed for leads, building timelines, etc.
- [Timeline Explorer](https://download.ericzimmermanstools.com/net9/TimelineExplorer.zip) (analyze the parsed artifacts)
- [MFTECmd](https://github.com/EricZimmerman/MFTECmd) (extract and parse MFT and USN Journal)
- [PECmd](https://github.com/EricZimmerman/PECmd) (Prefetch parser)
- [Registry Explorer](https://ericzimmerman.github.io/#forensic-tools) (parser and viewer, fixes dirty hives)
- [AppCompatCacheParser](https://github.com/EricZimmerman/AppCompatCacheParser) (shimcache, amcache parser)
- [JLECmd](https://github.com/EricZimmerman/JLECmd) (jumplist parser)
- [LECmd](https://github.com/EricZimmerman/LECmd) (parser for shortcut files)

``` powershell
# we run KAPE to extract targets and process modules
PS E:\KAPE> .\kape.exe --tsource C:\ --tdest E:\kapeoutput\targets --target !BasicCollection --tflush --mdest E:\kapeoutput\modules --module !EZParser --mflush
System info: Machine name: TEST, 64-bit: True, User: test1 OS: Windows10 (10.0.26200)
Using Target operations
  Flushing target destination directory E:\kapeoutput\targets
  Creating target destination directory E:\kapeoutput\targets
Found 12 targets. Expanding targets to file list...
Found 1,070 files in 4.185 seconds. Beginning copy...
  Deferring C:\Windows\System32\winevt\logs\Application.evtx due to IOException...
  Deferring C:\Windows\System32\winevt\logs\HardwareEvents.evtx due to IOException...
[..]
Deferred file count: 257. Copying locked files...
  Copied deferred file C:\Windows\System32\winevt\logs\Application.evtx to E:\kapeoutput\targets\C\Windows\System32\winevt\logs\Application.evtx. Hashing source file...
[..]
Copied 930 (Deduplicated: 140) out of 1,070 files in 780.0411 seconds. See E:\kapeoutput\targets\2026-06-23T12_53_52_9931707_CopyLog.csv for copy details

Using Module operations
  Setting --msource to E:\kapeoutput\targets since --msource was not provided
  Flushing module destination directory E:\kapeoutput\modules
  Creating module destination directory E:\kapeoutput\modules
  Module AmcacheParser: Found 1 processor
    Found processor Executable: AmcacheParser.exe, Cmd line: -f %sourceFile% --csv %destinationDirectory% -i --mp, Export: csv, Append: False!
[..]
  Module MFTECmd: Found 4 processors
  Module MFTECmd_$Boot: Found 2 processors
  Found processor Executable: MFTECmd.exe, Cmd line: -f %sourceFile% --csv %destinationDirectory%, Export: csv, Append: False!
  Module MFTECmd_$MFT: Found 2 processors
    Found processor Executable: MFTECmd.exe, Cmd line: -f %sourceFile% --csv %destinationDirectory%, Export: csv, Append: False!
  Module MFTECmd_$J: Found 2 processors
    Found processor Executable: MFTECmd.exe, Cmd line: -f %sourceFile% --csv %destinationDirectory%, Export: csv, Append: False!
[..]
Discovered 18 processors to run
Executing modules with file masks...
  Running AmcacheParser.exe: -f "E:\kapeoutput\targets\C\Windows\appcompat\Programs\Amcache.hve" --csv E:\kapeoutput\modules\ProgramExecution -i --mp
  Running AppCompatCacheParser.exe: -f "E:\kapeoutput\targets\C\Windows\System32\config\SYSTEM" --csv E:\kapeoutput\modules\ProgramExecution
  Running MFTECmd.exe: -f "E:\kapeoutput\targets\C\$Boot" --csv E:\kapeoutput\modules\FileSystem
  Running MFTECmd.exe: -f "E:\kapeoutput\targets\C\$MFT" --csv E:\kapeoutput\modules\FileSystem
[..]
Executed 18 processors in 805.5887 seconds

Total execution time: 1,585.7123 seconds
```

![ez](../media/ez.png) <br />
_image source: `ericzimmerman.github.io`_

### [LiME](https://github.com/jtsylve/LiME)

Loadable Kernel Module (LKM) for volatile memory acquisition from unix like devices, including Android.

It has a minimal interaction with user and kernel space processes during acquisition, producing forensically sound memory captures.

```bash
# we acquire memory over the network from the target to a collector
# on the target we find the kernel release
$ uname -r
6.17.0-35-generic
# on the collector we generate a .ko for the kernel release of our target
$ sudo apt install -y linux-headers-6.17.0-35-generic
$ git clone https://github.com/jtsylve/LiME.git
$ cd LiME/src/
$ make KVER=6.17.0-35-generic
$ ls
[..] lime-6.17.0-35-generic.ko
$ modinfo lime-6.17.0-35-generic.ko | grep vermagic
vermagic:       6.17.0-35-generic SMP preempt mod_unload modversions
# we transfer the .ko from the collector to the target
# on the target we insert the module and send the output over the network
$ sudo insmod ./lime-$(uname -r).ko path=tcp:4444 format=lime
# on the collector
$ nc 192.168.181.100 4444 > ram.lime
$ ls -l ram.lime
-rw-r--r-- 1 dfir dfir 4294367360 Jun 17 10:30 ram.lime
```

To analyze the linux memory images with Volatility, we need to the ISF (symbol table) for the target kernel.

To be discussed in the memory analysis lesson.

### [AVML](https://github.com/microsoft/avml)

AVML is another memory acquisition tool for unix devices, which can be used not knowing the kernel release.

``` bash
# we acquire memory over the network from the target to the collector
# on the collector we start a listener
$ nc -l -p 4444 > avml.mem
# on the target we create a fifo pipe
$ sudo mkfifo /tmp/mem.pipe
$ sudo chmod 666 /tmp/mem.pipe
# we start a background job to write from the pipe to the network to our collector
$ nc 192.168.181.200 4444 < /tmp/mem.pipe &
# we run avml and write to the pipe
$ sudo ./avml /tmp/mem.pipe
# on the collector we check the output
$ ls -l avml.mem
-rw-r--r-- 1 dfir dfir 3222923326 Jun 17 15:14 avml.mem
```

The symbol table is needed as well to analyze the memory capture with Volatility.

### [WinPmem](https://github.com/Velocidex/WinPmem/tree/master)

Apart from FTK Imager, WinPmem could also be used to capture Windows memory but only if DSE (Driver Signature Enforcement) is disabled already.

Disabling it would require a reboot, which defeats the purpose of capturing volatile memory.

``` powershell
# we run winpmem and write the output to a separate disk
PS E:\> .\winpmem64.exe E:\winpmem.raw
WinPmem64
Extracting driver to C:\Users\test1\AppData\Local\Temp\win5771.tmp
Driver Unloaded.
Loaded Driver C:\Users\test1\AppData\Local\Temp\win5771.tmp.
Deleting C:\Users\test1\AppData\Local\Temp\win5771.tmp
The system time is: 17:43:01
Will generate a RAW image
 - buffer_size_: 0x1000
CR3: 0x00001AE000
 6 memory ranges:
Start 0x00002000 - Length 0x0009E000
[..]
```

To summarize host-based acquisition:

For a live Windows system, `FTK Imager` handles both disk and memory.
`KAPE` is a better option when the time is short and you need the most important artifacts fast, before full imaging completes.

For a live Unix system, `UAC` collects the standard artifact set, while for memory acquisition `LiME` or `AVML` can be used to dump memory to a file or over the network.

For offline aquisition (when the disk is removed and connected to the forensic workstation with a write blocker) `dd` is used when interoperability is important, or `FTK Imager` when you need built-in compression, hash verification and case metadata.

## Evidence acquisition - Network

### [tcpdump](https://www.tcpdump.org/)

Command-line packet-capture and inspection tool, with powerful filtering capabilities.<br />
Used to capture traffic on a host during an investigation, or quickly inspecting a pcap without a GUI.<br />
Usage:

``` bash
# Capture all traffic on interface eth0
sudo tcpdump -i eth0
# Capture all traffic on interface eth0 on port 80
sudo tcpdump -i eth0 port 80
# Capture all traffic on interface eth0 on port 80 with no name or port resolution
sudo tcpdump -i eth0 port 80 -n
# Read traffic from a file with no name or port resolution
sudo tcpdump -nr file.pcap 
```

![tcpdump](../media/tcpdump.png)<br />

### [Wireshark](https://www.wireshark.org/download.html)

GUI packet-capture and inspection tool, allows analysis in a graphical interface, with shortcomings when it comes to large file sizes (it must load the entire .pcap file in memory). <br />

![wireshark](../media/wireshark.png) <br />

### [Arkime](https://github.com/arkime/arkime)

Arkime is an open-source network analysis and packet capture system that allows large-scale operations.
Can be deployed on multiple systems and handle tens of GB/sec.
3 main components:

- capture
- viewer
- opensearch / elasticsearch

The Viewer includes the Sessions page overview, listing individual sessions that can be expanded to view metadata and packet details that have been captured.

![Arkime](../media/arkime.png)<br />
_image source: [arkime](https://github.com/arkime/arkimeweb/blob/main/assets/sessions.png)_

## Evidence acquisition - Remote

### [Velociraptor](https://github.com/Velocidex/velociraptor)

Tool for collecting host based state information remotely via agents deployed on hosts. <br />
Typical deployment: <br />
![velociraptor](../media/velociraptor.png) <br />
_image source: `docs.velociraptor.app`_

## Time normalization

- time zones may differ on evidence sources in the investigation
- normalize to UTC for correlation
- keep conversion logs
- never overwrite original evidence

## Summary

![evidence-acquisition](../media/evidence-acquisition.png)

- acquire data from hosts and network devices, local or remote
- prioritize by volatility
- preserve evidence integrity
- normalize time to UTC

## Drills

### before-it-disappears

A service seems to have crashed unexpectedly, leaving behind a dump and a binary.
Could you reconstruct what the program was doing?

### broken-seal

We received a collection of KAPE triage artifacts, so we must follow procedure before we touch anything.

### droopy

In the same set of artifacts from broken-seal, the attackers tried to cover their tracks but left behind the exact source of the payload.
The flag format is SSS{ip:port/something}

### ospaperleak

This was a clumsy attacker.
They dropped all sorts of payloads among which a 32 bit one, see if you can find it.
The flag format is SSS{SHA1}

## Further practice

- capture a unix disk image with dd
- collect triage artifacts from a unix machine with UAC
- capture a unix memory image with AVML or LiME
- capture a windows disk and memory image with FTK Imager
- collect triage artifacts from a windows machine with KAPE
- capture traffic with tcpdump or wireshark

## References & further reading

[+] ISACA Whitepaper - [Overview of Digital Forensics](https://www.isaca.org/resources/white-papers/overview-of-digital-forensics)<br />
[+] IETF [RFC 3227 - Guidelines for Evidence Collection and Archiving](https://www.ietf.org/rfc/rfc3227.txt)<br />
[+] [Disk image formats](https://forensics.wiki/disk_images/)<br />
[+] Hal Pomeranz's [Linux Forensics](https://archive.org/details/HalLinuxForensics)<br />
[+] UAC - [Unix-like Artifacts Collector](https://github.com/tclahr/uac)<br />
[+] [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)<br />
[+] [EZ Tools](https://ericzimmerman.github.io/)<br />
[+] [tcpdump](https://www.tcpdump.org/)<br />
[+] [Wireshark](https://www.wireshark.org/download.html)<br />
[+] [Velociraptor](https://github.com/Velocidex/velociraptor)<br />
[+] [Artifact collection matrix](https://github.com/swisscom/ArtifactCollectionMatrix)<br />
[+] [Date and Time on the internet: Timestamps](https://datatracker.ietf.org/doc/html/rfc3339)<br />
[+] [A study of Forensic Imaging in the Absence of Write-Blockers](https://commons.erau.edu/cgi/viewcontent.cgi%3Farticle%3D1027%26context%3Ddb-security-studies&ved=2ahUKEwjl4YPEjMqUAxUcSvEDHW3QKO04ChAWegQIFhAB&usg=AOvVaw1c_U3hrxOQxFcrMIbGENSC)<br />
[+] [Introducing LECmd](https://binaryforay.blogspot.com/2016/02/introducing-lecmd.html)<br />
