# Solution

1. What is the attacker's user agent?

The flag format is `SSS{user-agent-string}`

``` bash
# we extract the user agents from the pcap with tshark
$ tshark -nnr thor.pcap -Y 'http.user_agent' -T fields -e 'http.user_agent'
Mozilla/4.08 (Charon; Inferno)
[..]
# we reconstruct the flag
SSS{Mozilla/4.08 (Charon; Inferno)}
```

2. What is the victim's username and hostname?

The flag format is `SSS{username-hostname}`

``` bash
# we extract the http data field and notice a user and hostname in the payload
$ tshark -nnr thor.pcap -Y 'http' -T fields -e data.data | head -n 1 | xxd -r -p
'ckav.ru"benjamin.franklinHIGHASAKITE-PCHIGHASAKITE-PC�k0245F45C0158529E7CB32E35Eaedcm'
[..]
# OR we notice it in the SMB Session Setup AndX Request
# in the wireshark GUI packet 10
# OR we check the packet with the weird user agent (583)
# we notice the username and hostname in the payload
12002700000007000000636b61762e7275010022000000620065006e006a0061006d0069006e002e006600720061006e006b006c0069006e0001001c00000048004900470048004100530041004b004900540045002d005000430001001c00000048004900470048004100530041004b004900540045002d005000430000050000d0020000010000000000060001000100c62b010001000000000000000000000001003000000032003400350046003400350043003000310035003800350032003900450037004300420033003200450033003500450005000000647674393900000000
# we reconstruct the flag
SSS{benjamin.franklin-HIGHASAKITE-PC}
```

3. What NTLM hash did the user authenticate with?

The flag format is `SSS{hash}`

``` bash
# we check packet 10 identified previously
# we notice the user authenticated with the Global NT Password
Expert Info (Chat/Security): NTLMv2 authenticated using <Global NT Password> (31d6cfe0...)
# we reconstruct the flag 
SSS{31D6CFE0D16AE931B73C59D7E0C089C0}
```

4. What is the DNS computer name?

The flag format is `SSS{dnscomputername}`

``` bash
# we extract the dns computer name from smb
$ tshark -nnr thor.pcap -Y 'smb' -T fields -e ntlmssp.challenge.target_info.dns_computer_name | awk 'NF'
vps37756.hostsailor.com
[..]
# we reconstruct the flag 
SSS{vps37756.hostsailor.com}
```
