# Investigation: persistence & privesc

Some of the questions we are answering in the case of persistence and privilege escalation are quite straightforward:

- what did the attacker do?
- were **backdoors** or **persistence mechanisms** installed?

**Persistence** is used by attackers so they don't need to repeat exploitation steps. <br />
After initial access, an attacker must take several steps before they can achieve their goal. <br />
At any point their **connection to the victim can be interrupted**. <br />
To avoid having to repeat the initial exploitation steps (maybe crafting a new phishing campaign, or using leaked credentials that may have changes), they typically establish some form of persistence. <br />

![Persistence](../media/persistence.png)<br />
*image source: https://pberba.github.io/security/*

In order to successfully establish a persistence mechanism, elevated privileges are usually, but not always, required. <br />
However, if the user that clicked the malicious file is privileged, or the exploited app or service is already running as root, administrator, or another privileged service user, there is no need for **privilege escalation**.

## Prerequisites

- a unix forensic vm
- a windows forensic vm

## Persistence

- boot or logon autostart execution / initialization scripts
  - new [registry run keys / startup folder](https://attack.mitre.org/techniques/T1547/001/)
  - loadable kernel modules (LKMs)
  - logon scripts, RC scripts
- scheduled tasks / jobs
- account creation
- account manipulation
  - additional users, roles or groups
  - new ssh authorized keys
  - new device registrations (ex. for MFA)
- create or modify system process
- event triggered execution
  - WMI subscriptions, COM hijacking, accessibility features, app shimming
  - shell configs, traps
- external remote services (ex. VPNs)

## Privilege Escalation

- abuse elevation control mechanism
  - setuid, setgid
  - bypass UAC
- access token manipulation
  - token impersonation / theft
  - ppid spoofing
- exploitation (ex. BYOVD)
- process injection

## Investigate case

- possible memory artifacts:
  - WMI persistence: unusual child processes of `wmiprvse.exe`
  - Registry Run keys: `Run` or `RunOnce` loaded in `HKLM` or `HKCU`
  - Scheduled tasks:
    - unusual child of `taskeng.exe` or `svchost.exe`
    - user or system-wide `cron` jobs in `/var/spool/cron/` or `/etc/crontab` or `/etc/cron.d/`
    - other utilities such as `fcron`, `anacron`, `tsp`, `at`, `batch` may be used for one-time or recurring jobs
  - suspicious dll loaded into `explorer.exe`, `svchost.exe`, `winlogon.exe`, `userinit.exe`
- possible disk artifacts:
  - Scheduled tasks: in `C:\Windows\System32\Tasks`
  - Registry Run keys: `Run` or `RunOnce` loaded in `HKLM` or `HKCU`
    - `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
    - `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`
    - `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
    - `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
  - Registry winlogon keys: `Winlogon\Userinit`, `Winlogon\Shell`, `Winlogon\Notify`
  - New service entry in `HKLM\SYSTEM\CurrentControlSet\Services`
  - Malicious startup files for all users and current user
    - `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup`
    - `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`
  - boot or logon initialization scripts in `/etc/init.d/`, `/etc/rc.local`, `/etc/update-motd.d`
  - shell configs in [FILES](https://manpages.debian.org/stretch/bash/bash.1.en.html#FILES)
  - systemd services or timers in `/etc/systemd/system`,`/run/systemd/system` and others
    - systemd [generators](https://manpages.debian.org/testing/systemd/systemd.generator.7.en.html)
  - existing user account creation or legitimate account modification
    - `lastlog` or `/var/log/auth.log` may show unexpected logins
    - account tampering is not needed if an attacker gains access to private ssh keys or valid, cleartext credentials

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

[+] [Understanding systemd services](https://wiki.debian.org/systemd/Services)<br />
[+] [Primer on persistence mechanisms](https://www.elastic.co/security-labs/primer-on-persistence-mechanisms)<br />
[+] [Persistence via systemd generators](https://pberba.github.io/security/2022/02/07/linux-threat-hunting-for-persistence-systemd-generators/)<br />
