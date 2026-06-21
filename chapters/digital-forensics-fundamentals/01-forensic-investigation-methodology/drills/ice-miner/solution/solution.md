# Solution

``` bash
# strings and binwalk reveal nothing useful
# we look at the file with xxd
$ xxd diamond_model.dot
# we notice an IHDR and an IEND after the dot file
[..]
00000340: 0a7d 0a89 504f 470d 0a1a 0a00 0000 0d49  .}..POG........I
00000350: 4844 5200 0001 e000 0000 7810 0200 0000  HDR.......x.....
[..]
00002930: 398d ebd2 0000 0000 4945 4e44 ae42 6082  9.......IEND.B`.
# the IHDR and IEND are typical of a PNG file
# the hints tell us of something mangled and botched
# we flip the magic bytes to PNG with hexedit
$ hexedit diamond_model.dot
[..]
00000340: 0a7d 0a89 504e 470d 0a1a 0a00 0000 0d49  .}..PNG........I
[..]
# binwalk now sees the png and extracts it
$ binwalk -e diamond_model.dot
DECIMAL       HEXADECIMAL     DESCRIPTION
835           0x343           PNG image, 480 x 120, 16-bit/color RGB, non-interlaced
876           0x36C           Zlib compressed data, best compression
# or extract it manually based on the binwalk offset
$ dd if=diamond_model.dot of=ddcarved.png bs=1 skip=835
# or
$ tail -c +836 diamond_model.dot > extracted.png
# open in an image viewer to see the flag
SSS{chUnK_0f_coaL}
```
