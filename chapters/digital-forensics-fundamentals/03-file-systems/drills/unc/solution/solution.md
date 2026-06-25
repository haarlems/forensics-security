# Solution

``` bash
# we analyze the $MFT file and filter for alternate data streams
# we find a deleted doc with Zone Id Contents
Zone Id Contents
"[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://unece.org/fileadmin/DAM/trade/agr/meetings/ge.01/document/2005_i05_apples_copacogeca.doc?__cf_chl_tk=PXpcd_3p2w8vXuiQI__7Wpv985YIus.EReurjxE7ckQ-1780931618-1.0.1.1-rCjOIYKKLGWmTklMkXkXqAl8n.qukpjoca3C8Xy1dns
HostUrl=https://unece.org/fileadmin/DAM/trade/agr/meetings/ge.01/document/2005_i05_apples_copacogeca.do"
# we build the flag
SSS{unece.org}
```
