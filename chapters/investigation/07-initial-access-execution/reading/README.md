# Investigation: initial access & execution

Here are the most frequently asked questions of the forensic investigator.<br />

- what happened?
  - what type of incident this is (ransomware, data exfil, insider threat, etc.)
- when did the breach first occur?
  - how was it detected?
  - is the attack ongoing?
  - what is the earliest indicator of compromise? (dwell time)
- how did the attackers get in?
  - initial access (phishing, web app exploit, exposed login portal, etc.)
  - known vulnerabilities exploited?
  - were credentials stolen or bruteforced?
  - was MFA bypassed?
- what systems are affected?
  - which endpoints, server, cloud resources are compromised?
  - are backups impacted?
  - are domain controllers affected?
    - did the attacked gain administrative or domain-level access?
  - what data is at risk?
- what did the attacker do?
  - was data accessed, exfiltrated, or altered?
  - were backdoors or persistence mechanisms installed?
  - was lateral movement observed?
  - were logs deleted or tampered with?
- who is responsible?
  - known [threat actor](https://attack.mitre.org/groups/)? ransomware gangs, nation-state activity, hacktivism?
  - insider involvement?
- what must be done immediately?
  - should credentials be reset?
  - should systems be isolated?
  - should law enforcement or regulators be notified?
  - should Incident Response plans be triggered?

In this chapter we will focus on answering the questions related to Initial Access and Execution.<br />

## Prerequisites

- a unix forensic vm
- a windows forensic vm

## ATT&CK Matrix

![MITRE](../media/mitre.png)

_NOTE: the MITRE ATT&CK Matrix is to be taken as a mere frame of reference._

## Initial access vectors

- [phishing](https://attack.mitre.org/techniques/T1566/) - the most common
  - targeted or non-targeted
- [valid accounts](https://attack.mitre.org/techniques/T1078/)
  - attackers don't break in, they log in
  - default accounts
  - credential stuffing from previous breach data
- [exploit public-facing application](https://attack.mitre.org/techniques/T1190/)
  - web apps, APIs (see [OWASP Top 10](https://owasp.org/Top10/2025/))
  - SSH, exposed ESXi OpenSLP services, exposed VMware vCenter servers
- [external remote services](https://attack.mitre.org/techniques/T1133/)
  - VPNs, Citrix
  - WinRM, VNC
  - Docker API, Kubernetes API server, kubelet, Kubernetes dashboard
- [trusted relationship](https://attack.mitre.org/techniques/T1199/)
  - third parties with delegated permissions
- [supply chain compromise](https://attack.mitre.org/techniques/T1195/)
  - software dependencies
  - CI/CD pipeline components (ex. GitHub Actions)
- and others

### Network

- identify connections to exposed services (VPN, SSH, RDP, HTTP/HTTPS)
- DNS requests to unusual domains
- connections not preceded by a DNS query, regular users rarely ever access a resource directly by IP
- unusual inbound connections, high volume of failed logins
- unexpected protocol versions or sequence anomalies
- http status manipulation where 4xx errors still deliver payloads
  - `cat http.log | zeek-cut method host uri user_agent status_code response_body_len`

## Execution

- [user execution](https://attack.mitre.org/techniques/T1204/)
  - social engineering to execute malicious code or open a malicious document or link
  - enabling remote access tools (RAT)
  - running malicious JavaScript in the browser that steals session cookies
- [command and scripting interpreter](https://attack.mitre.org/techniques/T1059/)
  - cli interfaces (unix shells, powershell, cmd), python, visual basic, JavaScript
  - used to execute initial access payloads
- [exploitation for client execution](https://attack.mitre.org/techniques/T1203/)
  - exploit software vulnerabilities in client applications
  - browser-based, office-applications, third-party applications (ex. Adobe Reader)
- and others

### Known Exploited Vulnerabilities

The [KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) database is a list of CVEs that are confirmed to be exploited in the wild.<br />

- helps narrow down initial access hypotheses: phishing, credential abuse or _exploitation of a vulnerability_?
- vulnerabilities in: edge devices, web apps, email servers, app frameworks
- cross-reference exposed services against KEV CVEs
  - _ex. if the victim has exposed Atlassian servers, worth checking if any new KEVs_
- search for exploitation artifacts tied to specific CVEs

## Investigate case

- possible memory artifacts:
  - suspicious child processes of `outlook.exe`, `chrome.exe`, `winword.exe`, `excel.exe`
  - office spawning `powershell.exe`, `cmd.exe`, `rundll32.exe`, `mshta.exe`, `wscript.exe`
- possible disk artifacts:
  - malicious attachments in user writable locations like Downloads, AppData, or Temp
  - browser history for phishing URL
  - web cache (cached phishing page), amcache, shimcache
  - evidence of file access in $MFT, prefetch files for `cmd.exe` or `powershell.exe`
  - Event ID 4688 A new process has been created from the Windows Security Log
- possible network artifacts:
  - `SMTP` session with suspicious or spoofed sender, lookalike or newly registered domain
  - `DNS` query to phishing domain
  - `TLS` handshake to phishing server
  - `HTTP` `GET`, `POST` if credentials submitted

## Summary

- initial access exploitation, stolen credentials to exposed remote services, phishing
- execution frequently involves user execution via social engineering
- investigate in memory, on disk and network capture looking for specific indicators

### red

Something weird was happening on this machine, sift through the triage artifacts to find what happened.

## Further reading

[+] [Kill chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)<br />
[+] [MITRE ATT&CK Matrix](https://attack.mitre.org/)<br />
[+] [Malware families](https://malpedia.caad.fkie.fraunhofer.de/families)<br />
[+] [Threat actor groups](https://attack.mitre.org/groups/)<br />
[+] [OWASP Top 10](https://owasp.org/Top10/2025/)<br />
[+] [Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)<br />
