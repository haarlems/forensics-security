
# Solution

``` bash
# exiftool shows us the first 2 parts of the flag
$ exiftool evidence.png
ExifTool Version Number         : 12.16
File Name                       : evidence.png
Directory                       : .
File Size                       : 24 KiB
File Modification Date/Time     : 2026:06:16 11:22:07+00:00
File Access Date/Time           : 2026:06:19 14:59:07+00:00
File Inode Change Date/Time     : 2026:06:19 15:00:50+00:00
File Permissions                : rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 800
Image Height                    : 600
Bit Depth                       : 16
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Comment                         : X3lyM3Zle1NTUw==
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Artist                          : X3NldjQzbF90YzR0bjBj
Y Cb Cr Positioning             : Centered
Warning                         : [minor] Trailer data after PNG IEND chunk
Image Size                      : 800x600
Megapixels                      : 0.480

# strings shows us some hex after the IEND footer
$ strings evidence.png
[..]
IEND
615f74723463655f736f6d3377483372657d

# we revert it with xxd
$ strings evidence.png | tail -1 | xxd -r -p
a_tr4ce_som3wH3re}

# we reassemble the flag
$ { for f in PNG:Comment EXIF:Artist; do exiftool -s3 -$f evidence.png | base64 -d | rev; done; strings evidence.png | tail -1 | xxd -r -p; } | tr -d '\n'; echo
SSS{ev3ry_c0nt4ct_l34ves_a_tr4ce_som3wH3re}
```
