# Solution

``` bash
# we check the hashes against the artifacts
$ sha256sum -c hashes.txt | grep FAILED
C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker Encrypt All Drives: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
# we check the failed one
$ file C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker\ Encrypt\ All\ Drives
C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker Encrypt All Drives: XML 1.0 document, Little-endian UTF-16 Unicode text, with CRLF line terminators
# we analyze the xml and find an interesting action
$ iconv -f UTF-16 -t UTF-8 C/Windows/system32/Tasks/Microsoft/Windows/BitLocker/BitLocker\ Encrypt\ All\ Drives
[..]
  </Actions>
  <Actions HashIntegrity="failed">
          <ClassId>%%%M:dEx1HcD;va5d&;`@O</ClassId>
  </Actions>
# now we decode the value with standard ROT47
# and we get a familiar string
$ echo '%%%M:dEx1HcD;va5d&;`@O' | python3 -c "import sys; print(''.join(chr(33+(ord(c)-33+47)%94) if 33<=ord(c)<=126 else c for c in sys.stdin.r
ead().rstrip(chr(10))))"
TTT|i5tI`w4sjG2d5Uj1o~
# we shift to 46 and get the flag
$ echo '%%%M:dEx1HcD;va5d&;`@O' | python3 -c "import sys; print(''.join(chr(33+(ord(c)-33+46)%94) if 33<=ord(c)<=126 else c for c in sys.stdin.r
ead().rstrip(chr(10))))"
SSS{h4sH_v3riF1c4Ti0n}
```
