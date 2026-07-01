# Solution

```bash
# we check the bashrc config and find an odd addition:
$ cat \[root\]/home/ritabeakerlab/.bashrc
# do not delete # admin #
(/tmp/.sysadmin/debian_chroot/sysrun.sh >/dev/null &)
# we reconstruct the flag with the full name of the file
Flag SSS{/tmp/.sysadmin/debian_chroot/sysrun.sh}
```
