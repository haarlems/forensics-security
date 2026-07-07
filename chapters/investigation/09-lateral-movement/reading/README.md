# Investigation: lateral movement, defense evasion & credential access

Some of the questions we are answering:

- what systems are affected?
  - which endpoints, server, cloud resources are compromised?
  - are backups impacted?
  - are domain controllers affected?
    - did the attacked gain administrative or domain-level access?
  - what data is at risk?
- what did the attacker do?
  - was data accessed, exfiltrated, or altered?
  - was lateral movement observed?
  - were logs deleted or tampered with?

## Prerequisites

- a unix forensic vm
- a windows forensic vm

## Lateral Movement

- remote services
  - RDP, SSH, SMB, WinRM
  - via valid accounts or via service exploitation
- alternate authentication material
  - pass the hash (PTH), pass the ticket (PTT)
- internal phishing
- lateral tool transfer
  - admin shares, RDP
  - scp, rsync, `curl`, sftp, onedrive

## Defense Evasion

- impair defenses
  - disable logging, EDR
  - disable or modify firewall
  - downgrade attacks
- indicator removal
  - clear logs, cmd history
  - timestomping
  - delete files
- hide artifacts
  - users, files, directories, file systems, virtual machines
  - file path exclusions

## Credential Access

- OS credential dumping
  - LSASS, SAM, NTDS, DCSync
  - `/etc/passwd` and `/etc/shadow`
- bruteforce
  - password guessing
  - password cracking
  - password spraying
  - credential stuffing
- steal or forge certificates (AD CS)
- steal or forge Kerberos tickets
  - golden ticket, silver ticket
  - kerberoasting, asrep roasting
- unsecured credentials
  - in files, chats, shell history
  - in credential managers, web browsers
- network sniffing (ex. LLMNR poisoning)
- keylogging

## Investigate case

- possible memory artifacts:
  - todo
- possible disk artifacts:
  - todo
- possible network artifacts:
  - todo

## Summary

- summary

## Drills

### Challenge 1

Description

### Challenge 2

Description

### Challenge 3

Description

## Further reading

[+] [todo](todo)<br />
