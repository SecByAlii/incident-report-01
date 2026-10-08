# 6. Findings

The three questions the practice asked, answered from the evidence.

## 1. How did they get in? (root cause)

**A single internet-reachable account with a guessable password and no second factor.**

The attacker reached a Windows logon on `win01` and guessed the `analyst` account's password
by brute force — 7 attempts, then success, in under three seconds (timeline 12:06:01–12:06:03).
Three conditions had to all be true for that to work, and all three were:

1. **The login was reachable** from where the attacker was. An exposed remote login is the
   front door.
2. **The password was guessable** — weak or reused enough to fall to automated guessing in
   seconds.
3. **There was no multi-factor authentication.** With MFA, a guessed password alone gets the
   attacker nowhere. Without it, the password *is* the entire lock.

Root cause is not "a weak password" by itself — it's the combination. MFA alone would have
stopped this intrusion at 12:06:03 even with the password guessed.

## 2. What did they do? (scope and impact)

Once in, the attacker worked quickly and with clear intent to stay:

- **Reconnaissance** — enumerated the account's privileges, other local accounts, and the
  network (`whoami /all; net user; ipconfig /all`), hidden inside encoded PowerShell.
- **Three persistence mechanisms** — a Run key (`UpdateManager`), a backdoor administrator
  account (`helpdesk_svc`), and a SYSTEM scheduled task (`WindowsHealthMonitor`). Any one of
  them re-admits the attacker; together they're built to survive a partial cleanup.
- **A credential-theft attempt** — behavior consistent with trying to dump credentials from
  LSASS to move deeper into the network. This was **blocked by the host's LSA protection** and
  did not succeed.
- **Anti-forensics** — cleared the Security log to destroy the record.

**Scope:** one host (`win01`) and one compromised account (`analyst`), plus the attacker-created
`helpdesk_svc`. **Impact:** full administrative control of the machine and durable, hidden
re-entry. No evidence of data theft or lateral movement to other machines *in the window
observed* — but the recon and the LSASS attempt show that was the direction of travel. This
was caught early, in the setup phase, before it became a breach or ransomware.

## 3. Why did it work? (the gaps)

Mapped to the weaknesses that were already known to exist at a practice like this — several of
them flagged in the [HIPAA risk assessment](https://github.com/SecByAlii/hipaa-risk-assessment):

| What let it happen | The gap |
|--------------------|---------|
| Password guessed in seconds | Weak/guessable password; no account lockout after repeated failures |
| Guessed password = full access | **No multi-factor authentication** (the biggest single gap) |
| Remote login reachable to attackers | Exposed remote-access surface, not restricted |
| Attacker reached admin / SYSTEM | The everyday account had more privilege than it needed |
| Persistence planted unnoticed in real time | Detection existed, but no one was watching the alerts live |

The one thing that **did** work: LSA protection blocked the credential-dump, and real-time log
forwarding preserved the evidence. Those two controls are the reason this is a contained
incident with a complete record instead of a quiet, ongoing compromise.

## Bottom line

A weak, unprotected login let an attacker take full control of a machine and dig in three ways
in under three minutes. They were stopped from going further by one OS control (LSA protection)
and caught because the logs were off the box before they could be erased. The intrusion is
real, it's scoped to one host, and — critically — it was caught in its early stage, before
real damage.
