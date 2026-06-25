# Solution

``` powershell
# use KAPE to run modules against the targets
> .\kape.exe --msource "targets\" --mdest "modules\" --module !EZParser
# in modules/FileSystem we find the $MFT file
# we open it in Timeline Explorer and filter for .exe files
# we find a payload.exe with a different Created0x30 field
Line	Tag	Entry Number	Sequence Number	Parent Entry Number	Parent Sequence Number	In Use	Parent Path	File Name	Extension	Is Directory	Has Ads	Is Ads	File Size	Created0x10	Created0x30	Last Modified0x10	Last Modified0x30	Last Record Change0x10	Last Record Change0x30	Last Access0x10	Last Access0x30	Zone Id Contents	Reparse Target	Reference Count	SI<FN	u Sec Zeros	Copied
1388	Unchecked	1351	6	171279	1	Checked	.\Users\test1\Downloads	payload.exe	.exe	Unchecked	Checked	Unchecked	256000	2026-04-19 18:46:15	2026-06-10 16:38:27	2026-04-19 18:46:15	2026-06-10 16:38:27	2026-04-19 18:46:15	2026-06-10 16:38:27	2026-06-10 17:01:38	2026-06-10 16:38:27			1	Checked	Checked	Unchecked
# we build the flag based on the given format
SSS{2026-06-06-11:45:47}
```
