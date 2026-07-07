# Solution

1. Who stole it?

The flag format is `SSS{IP}`

``` bash
# we notice suspicious requests to whooptm.cyou
# which resolves to ip 
SSS{172.138.43.121}
```

2. How many bytes did the victim send to the attacker?

The flag format is `SSS{numberofbytes}`

``` bash
# we extract with tshark
$ tshark -nnr thief.pcap -q -z io,stat,0,'SUM(frame.len)frame.len && ip.src==10.1.20.101 && ip.dst==174.138.43.121'
===================================================================================
| IO Statistics                                                                   |
|                                                                                 |
| Duration: 663.0 secs                                                            |
| Interval: 663.0 secs                                                            |
|                                                                                 |
| Col 1: SUM(frame.len)frame.len && ip.src==10.1.20.101 && ip.dst==174.138.43.121 |
|---------------------------------------------------------------------------------|
|                |1        |                                                      |
| Interval       |   SUM   |                                                      |
|--------------------------|                                                      |
|   0.0 <> 663.0 | 5836953 |
# we reconstruct the flag 
SSS{5836953}
```

3. What language does the attacker presumably use?

The flag format is `SSS{language}`

``` bash
# in packet 263 we notice russian comments
// Преобразуем в URLSearchParams\r\n
# we reconstruct the flag
SSS{russian}
```
