# Solution

``` bash
# analyze the file with file, strings, xxd, binwalk
# extract the hidden file with binwalk -e
$ binwalk -e snack.bin
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
4046          0xFCE           gzip compressed data, from Unix, last modified: 1970-01-01 00:00:00 (null date)
$ ls _slack_snack.bin.extracted/
FCE  FCE.gz
$ cat _slack_snack.bin.extracted/FCE
SSS{sl4cK_sP4ce_h1dEs_s3creT5}
# or with dd
$ dd if=snack.bin bs=1 skip=4046 2>/dev/null | gunzip -c
SSS{sl4cK_sP4ce_h1dEs_s3creT5}
```
