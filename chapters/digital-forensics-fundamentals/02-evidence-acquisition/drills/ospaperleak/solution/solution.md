# Solution

``` powershell
# use KAPE to run modules against the targets
> .\kape.exe --msource "targets\" --mdest "modules\" --module !EZParser
# in modules/ProgramExecution we find the AmCache
> ls .\modules\ProgramExecution\20260618141646_Amcache_UnassociatedFileEntries.csv
Directory: [..]\modules\ProgramExecution
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         6/18/2026   2:16 PM          80879 20260618141646_Amcache_UnassociatedFileEntries.csv
# we open it in Timeline Explorer and find multiple entries
Line	Tag	Application Name	Program Id	File Key Last Write Timestamp	SHA1	Is Os Component	Full Path	Name	File Extension	Link Date	Product Name	Size	Version	Product Version	Long Path Hash	Binary Type	Is Pe File	Bin File Version	Bin Product Version	Language	Description
205	Unchecked	Unassociated	000671cc996f3d457c0a58efb1c266dd6c5a0000ffff	2026-04-19 18:36:49	2a45d8a3bdffb38a59a98a8ca3f73394195b0595	Unchecked	c:\users\test1\winpagedrip.exe	winpagedrip.exe	.exe	2026-03-16 11:50:43		1007616				pe32_i386	Unchecked			0	
103	Unchecked	Unassociated	0006163feb71bcbd6dc1413d27a99e49abac0000ffff	2026-04-19 18:36:45	a222267c5deb15003faf73949f936dbbf1c068a2	Unchecked	c:\users\test1\notepatch.exe	notepatch.exe	.exe	2026-03-16 11:51:42		903168				pe64_amd64	Unchecked			0	
13	Unchecked	Unassociated	000688f466beb6626fc165ed042fe4664be80000ffff	2026-04-19 18:15:02	37f0b70c5cb067da09d1705a83c88dd78ae05274	Unchecked	c:\users\test1\bitmorph.exe	bitmorph.exe	.exe	2026-03-16 11:51:42		903168				pe64_amd64	Unchecked			0	
# the challenge mentions a 32bit payload, so it must be
2a45d8a3bdffb38a59a98a8ca3f73394195b0595	winpagedrip.exe	pe32_i386
# we build the flag based on the format
SSS{2a45d8a3bdffb38a59a98a8ca3f73394195b0595}
```
