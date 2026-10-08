# Incident Report — Coquina Eye Care

**Incident:** Compromise of a Windows host (`win01`) via brute-forced credentials
**Prepared by:** Allison Grainger
**Classification:** Confirmed intrusion — contained, caught in early stage
**Status:** Remediation recommended (below)

---

## Executive summary

Over the weekend, an attacker broke into one of the practice's Windows computers by guessing
the password of a staff account. Within about three minutes of getting in, they set up three
separate hidden ways to get back in, attempted to steal more passwords from the machine, and
then deleted the computer's security logs to cover their tracks.

The good news, and it's significant: **this was caught early, and nothing was lost.** The
attempt to steal more passwords was blocked automatically by the computer's built-in
protection. And because the practice's logs are copied to a separate monitoring system the
moment they're written, the attacker's attempt to erase the evidence failed — the full record
survived, which is the only reason we can tell you exactly what happened.

The intrusion is limited to the one machine. There is no evidence the attacker reached other
computers or patient data in the time observed — but the way they behaved shows that was
their next step, so acting now matters.

**The single most important fix:** turn on multi-factor authentication for logins. With it,
guessing the password would not have been enough to get in, and this intrusion would have been
stopped at the front door.

---

## What happened, in brief

| | |
|---|---|
| **How they got in** | Guessed a staff account's password (7 tries, then success, in 3 seconds). No multi-factor authentication meant the password was the only lock. |
| **What they did** | Scouted the machine and network, planted three hidden ways back in (a startup entry, a fake "help desk" administrator account, and a scheduled task), and tried to steal more passwords. |
| **What stopped them going further** | The computer's built-in password protection blocked the password-theft attempt. The attack was caught before it spread or caused damage. |
| **Scope** | One machine (`win01`), one staff account compromised, plus one account the attacker created. No confirmed data loss. |

A full technical account is in the [timeline](04-timeline.md) and [findings](06-findings.md).

---

## Root cause

A single internet-reachable login with a **guessable password and no second factor**. Any one
of three fixes would have broken the attack; none were in place.

---

## Recommendations

In priority order. The first three are free or near-free and close the door this attacker
walked through.

| Priority | Action | Why |
|----------|--------|-----|
| **1** | **Turn on multi-factor authentication** for all staff logins | Stops this exact attack even if a password is guessed. Single highest-impact fix. |
| **2** | **Reset the compromised password and remove the attacker's access** | See [containment](07-containment-eradication.md). Reset `analyst`, remove the backdoor account and both other persistence mechanisms, ideally reimage the machine. |
| **3** | **Lock out accounts after repeated failed logins**, and restrict remote access | Automated guessing stops working when the account locks after a handful of failures. |
| **4** | **Give everyday accounts only the access they need** | The compromised account reached administrator/SYSTEM. Less standing privilege means less damage from one compromise. |
| **5** | **Have someone actually watch the alerts** | The monitoring caught this in real time — but over a weekend, no one looked for hours. Detection only helps if an alert reaches a human. |
| **6** | **Keep the two controls that saved you** | Built-in password (LSA) protection and real-time log forwarding are the reason this is a contained incident with a full record. Keep both on, everywhere. |

Several of these were already flagged as risks in the practice's
[HIPAA Security Risk Assessment](https://github.com/SecByAlii/hipaa-risk-assessment) — this
incident is what those risks look like when they're exercised.

---

## Conclusion

An attacker took full control of one machine and dug in, fast — but was blocked from spreading
by one automatic control and caught because the evidence was out of their reach. This is a
contained, early-stage intrusion, not a breach. Closing the gaps above — multi-factor
authentication first — turns "we caught one in time" into "this can't happen the same way
again," which is the whole goal.
