# Solution

``` powershell
# use KAPE to run modules against the targets
> .\kape.exe --msource "targets\" --mdest "modules\" --module !EZParser
# in modules/FileSystem we find the $MFT file
# we open it in Timeline Explorer and filter for .py files
# there are only two files with a Created0x30 field
Line	Tag	Entry Number	Sequence Number	Parent Entry Number	Parent Sequence Number	In Use	Parent Path	File Name	Extension	Is Directory	Has Ads	Is Ads	File Size	Created0x10	Created0x30	Last Modified0x10	Last Modified0x30	Last Record Change0x10	Last Record Change0x30	Last Access0x10	Last Access0x30	Zone Id Contents	Reparse Target	Reference Count	SI<FN	u Sec Zeros	Copied
371347	Unchecked	248478	4	248476	4	Checked	.\kapeoutput\targets\C\$Recycle.Bin\S-1-5-21-3757327896-2532397730-150904874-1001	$RB3CC5R.py	.py	Unchecked	Unchecked	Unchecked	966	2026-04-19 18:20:28	2026-06-06 11:45:47	2026-04-19 18:20:37	2026-06-06 11:45:47	2026-06-06 11:45:47	2026-06-06 11:45:47	2026-04-22 10:09:24	2026-06-06 11:45:47			1	Checked	Unchecked	Unchecked
371346	Unchecked	248477	4	248476	4	Checked	.\kapeoutput\targets\C\$Recycle.Bin\S-1-5-21-3757327896-2532397730-150904874-1001	$IB3CC5R.py	.py	Unchecked	Unchecked	Unchecked	94	2026-06-05 16:29:35	2026-06-06 11:45:47	2026-06-05 16:29:35	2026-06-06 11:45:47	2026-06-06 11:45:47	2026-06-06 11:45:47	2026-06-06 10:35:11	2026-06-06 11:45:47			1	Checked	Unchecked	Unchecked
# both have the same timestamp, so we build the flag based on the format
SSS{2026-06-06-11:45:47}
```
