# Investigation: Operation Daylight Saving

## Prerequisites

- a unix forensic vm
- a windows forensic vm

## Identification and scoping

## Evidence collection

## Initial event analysis & preliminary correlation

## Event normalization, deconfliction & secondary correlation

## Timeline

### Building initial timelines

### Building a master timeline

### EZ Tools - Timeline Explorer

## Kill chain analysis

### Initial access

### Evidence of execution

### Internal discovery

### Persistence

### Lateral movement

### Defense evasion

### Credential harvesting

### C2

#### Identify C2 patterns on the network

- the act of a compromised system (client) checking in with the server for any commands is often referred to as **beaconing**
- if no commands received from the server, the client goes to sleep a set amount of time, called **sleep** time
- sleep time can vary a set % called **jitter** (ex. sleep 10s with jitter 20% -> sleep can be anything between 8-12s)
- beaconing creates regular traffic patterns that can be identified when looking at behaviour over a time range (ex. 24h)

Protocols used for C2 communication:

- HTTP/HTTPS = the most common, blends in with normal traffic that flows in an enterprise environment
- DNS = commonly used but suspicious traffic can be identified (abnormally large number of requests to the same domain, abusing [TXT, CNAME, NS, MX](https://www.activecountermeasures.com/a-network-threat-hunters-guide-to-dns-records/) records)
- ICMP = not very common as it's immediately suspicious to see large amounts of ICMP requests to remote hosts
- SMB = used to pivot peer-to-peer in windows environments, does not cross external network boundaries

With an additional layer of abstraction, C2s can also be achieved via applications, like the Google suite (Calendar, Drive, etc.), Discord, Velociraptor and many others, but they are outside the scope of this class.

### Exfiltration

#### Identify data exfiltration on the network

- large volumes of bytes seen outbound to suspicious destinations in zeek logs or packet captures
- outbound HTTP/S `POST` requests with large payloads or to unusual URLs
- unusual FTP/FTP usage
- unauthorized cloud storage uploads

### Diamond model

## Reporting

### Executive summary

### IOCs

### MITRE ATT&CK

## Summary

- summary

## Drills

### Challenge 1 - Operation Daylight Saving

Description

## Further reading

[+] [todo](todo)<br />
