# Solution

``` bash
# we check the .bash_logout and find a weird addition
$ cat \[root\]/home/ritabeakerlab/.bash_logout
[..]
# app cleanup routine executed at each logout
rm -rf /tmp/khsolutions-app-cache-$USER
[..]
# we reconstruct the flag
SSS{/tmp/khsolutions-app-cache-$USER}
```
