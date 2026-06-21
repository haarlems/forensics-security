# Solution

``` bash
# we disable the streams to see the plaintext that we could grep for
$ qpdf --qdf --object-streams=disable custody_sheet.pdf - | less
[..]
0 -200 Td
1 1 1 rg
(SSS{n0_0ne_r34ds_th3_f1n3_pr1nt}) Tj
ET
endstream
[..]

# or we use a tool that understands PDF to dump all the text
$ pdftotext custody_sheet.pdf - | grep SSS
SSS{n0_0ne_r34ds_th3_f1n3_pr1nt}

# or we use binwalk to extract the flag
$ binwalk custody_sheet.pdf
DECIMAL       HEXADECIMAL     DESCRIPTION
0             0x0             PDF document, version: "1.4"
301           0x12D           Zlib compressed data, default compression
$ binwalk -e custody_sheet.pdf
[..]
$ ls _custody_sheet.pdf.extracted/
12D  12D.zlib
$ cat _custody_sheet.pdf.extracted/12D | grep SSS
(SSS{n0_0ne_r34ds_th3_f1n3_pr1nt}) Tj
```
