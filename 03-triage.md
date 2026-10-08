# 3. Triage — where the investigation starts

Before reconstructing anything, the first job is to answer one question fast: **is this real,
or is it noise?** Most alerts are noise. An investigation only earns the hours it takes if
there's a genuine compromise behind the alert.

## The initial indicator

The alert that started this came from the SIEM over the weekend: a burst of **failed logons
on `win01`, immediately followed by a successful one**, then activity outside business
hours. In SIEM terms, the `soc_detections` logic flagged two things on the same host within
seconds:

- **Brute force** — 5+ failed logons in a short window (fired at 7 failures).
- **A successful logon inside that failed-logon burst** — the single most important
  early-warning signal there is.

## Why this isn't noise

A few failed logons are normal — people fat-finger passwords. Two things move this out of
"noise" immediately:

1. **The pattern.** Seven failures in the same second, then a success two seconds later, is
   not a human mistyping. It's automated guessing that landed.
2. **The timing.** It happened in the middle of the night, when no one at a 3-provider clinic
   is logging in.

That combination — *failures, then success, off-hours* — is the textbook signature of a
brute-forced account. It clears the bar for a full investigation on its own.

## Triage decision

**Treat as a confirmed intrusion and investigate fully.** The working assumption from this
point is that the `analyst` account is compromised and the attacker had hands-on access to
`win01`. The next step is to reconstruct exactly what they did with it — before, and after,
they got in.

## Evidence-handling note

The host was left powered on and untouched, and the investigation works from the
**forwarded logs in the SIEM**, not the live machine — because, as the timeline will show,
the attacker cleared the local Security log. The off-box copy is the authoritative record.
