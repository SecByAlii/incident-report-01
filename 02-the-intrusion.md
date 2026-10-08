# 2. The Intrusion (ground truth)

This document records **exactly what was done to the host**, so the investigation that
follows can be checked against it. In a real engagement you never have this — you
reconstruct it from evidence. Here it's written down on purpose, so a reader can see that
the timeline reconstructed in [Phase 4](04-timeline.md), built only from the logs, matches
what actually happened.

Everything below was run against `win01` (the lab stand-in for Coquina's Windows host) in a
single session. The techniques map to the same attack simulations documented in
[home-lab-soc](https://github.com/SecByAlii/home-lab-soc); here they're chained into one
realistic intrusion instead of tested in isolation.

> Lab-timestamp note: the evidence timestamps are the real lab run time, compressed into a
> few minutes. The scenario's "overnight, over a weekend" framing is narrative; the
> investigation uses the actual recorded times.

## Attacker objective

A credential-based intrusion of the kind that hits small businesses constantly: get in
through a weak login, establish durable access, try to steal more credentials to spread,
and erase the tracks. This is the early, pre-encryption stage of a typical ransomware
intrusion — exactly the window where catching it matters most.

## The kill chain

| # | Tactic | Technique | What the attacker did | Evidence it leaves |
|---|--------|-----------|------------------------|--------------------|
| 1 | Initial Access | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) Brute force | Guessed the password of the `analyst` account — many failures, then success | Security **4625** ×7, then **4624** (type 2) |
| 2 | Execution | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) PowerShell | Ran a Base64-encoded PowerShell recon command (`whoami /all`, `net user`, `ipconfig /all`) | Sysmon **1** with `-EncodedCommand` |
| 3 | Persistence | [T1547.001](https://attack.mitre.org/techniques/T1547/001/) Run key | Added an autostart Run key `UpdateManager` → `C:\Windows\Temp\upd.exe` | Sysmon **13** |
| 4 | Credential Access | [T1003.001](https://attack.mitre.org/techniques/T1003/001/) LSASS | Tried to open LSASS to dump credentials — **denied by LSA protection** | OpenProcess fails (handle 0, error 5); no dump |
| 5 | Persistence | [T1136.001](https://attack.mitre.org/techniques/T1136/001/) Local account | Created a backdoor admin account `helpdesk_svc` | Security **4720** + add-to-Administrators |
| 6 | Persistence | [T1053.005](https://attack.mitre.org/techniques/T1053/005/) Scheduled task | Registered `WindowsHealthMonitor` to run every 15 min as SYSTEM | Security **4698** |
| 7 | Defense Evasion | [T1070.001](https://attack.mitre.org/techniques/T1070/001/) Clear logs | Cleared the Security event log to destroy the evidence | Security **1102** (and the wiped local log) |

The one that didn't work — the LSASS credential dump (step 4) — is as important as the ones
that did. The attacker tried to steal more credentials to move deeper into the network and
was stopped at the OS layer. That failed attempt is still recorded, and it tells the
investigator the intruder was after credentials, not just a single machine.

## Reproduction

The full chain is reproducible. Initial access and the successful logon:

```powershell
# 7 failed logons for 'analyst', then one success (brute force)
Add-Type -Namespace X -Name L -MemberDefinition '[DllImport("advapi32.dll",SetLastError=true)]
public static extern bool LogonUser(string u,string d,string p,int lt,int lp,out IntPtr t);'
$t=[IntPtr]::Zero
1..7 | % { [X.L]::LogonUser("analyst",$env:COMPUTERNAME,"Autumn2024!$_",2,0,[ref]$t) }
[X.L]::LogonUser("analyst",$env:COMPUTERNAME,"<correct password>",2,0,[ref]$t)
```

Execution, persistence, credential-access attempt, and evasion:

```powershell
# Encoded PowerShell recon
$enc=[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes("whoami /all; net user; ipconfig /all"))
Start-Process powershell "-NoProfile -WindowStyle Hidden -EncodedCommand $enc" -Wait

# Run-key persistence
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v UpdateManager /t REG_SZ /d "C:\Windows\Temp\upd.exe" /f

# LSASS dump attempt (denied by LSA protection)
# OpenProcess(lsass, PROCESS_VM_READ|PROCESS_QUERY_INFORMATION) -> fails

# Backdoor admin account (New-LocalUser avoids net.exe's >14-char prompt)
New-LocalUser helpdesk_svc -Password (ConvertTo-SecureString "W1nterGarden2024" -AsPlainText -Force) -AccountNeverExpires
Add-LocalGroupMember -Group Administrators -Member helpdesk_svc

# Scheduled-task persistence
schtasks /create /tn "WindowsHealthMonitor" /tr "cmd.exe /c powershell -NoP -W Hidden -Command exit" /sc minute /mo 15 /ru SYSTEM /f

# Cover tracks
wevtutil cl Security
```

## What the attacker got wrong

They cleared the Security log (step 7) to destroy the record — and on the local machine,
it worked. But the host was forwarding its logs to a SIEM in real time, so every event was
already off the box before it was wiped. The attacker destroyed the copy they could reach
and left the one that mattered untouched. **That single design choice — forward logs
off-host as they're written — is why this investigation is possible at all**, and it's the
first thing the findings will credit.
