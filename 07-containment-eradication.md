# 7. Containment, Eradication, and Recovery

Answering the practice's third question — *is it over, and how do we make sure?* — in the
order a responder actually does it: stop the bleeding first, remove the attacker second,
confirm clean third.

## Immediate containment (do first, in minutes)

Cut off access before cleaning up, so the attacker can't react while you work.

1. **Isolate `win01` from the network.** Pull it off the LAN (disable the network adapter or
   unplug it) so any live session or callback is severed, but leave it powered on to preserve
   evidence.
2. **Disable both the compromised and the backdoor accounts** — don't delete yet:
   - `analyst` (compromised) and `helpdesk_svc` (attacker-created).
   - Disabling first, deleting later, keeps them as evidence while killing access.
3. **Force-reset the `analyst` password** (and any account that shared it). Assume the
   password is known to the attacker and burned.

## Eradication (remove every foothold)

Remove all three persistence mechanisms — miss one and the attacker is back.

| Remove | Command |
|--------|---------|
| Backdoor admin account | `Remove-LocalUser helpdesk_svc` (after preserving it as evidence) |
| Scheduled task | `schtasks /delete /tn WindowsHealthMonitor /f` |
| Run key | `reg delete "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v UpdateManager /f` |
| Dropped payload | delete `C:\Windows\Temp\upd.exe` if present |

Then **hunt for what you haven't found yet.** An attacker who planted three persistence
mechanisms may have planted a fourth. Check for other new accounts, other scheduled tasks,
services, WMI subscriptions, and other autostart locations before declaring the host clean.

## Recovery

- **Rebuild over clean.** Because the attacker had SYSTEM and cleared logs, the most
  defensible path is to **reimage `win01` from a known-good build** rather than trust a
  cleanup. Eradication by hand is for when reimaging isn't possible; a flattened and rebuilt
  machine is the only way to be certain nothing was missed.
- **Restore service** only after the account and access gaps (below and in
  [recommendations](08-report.md)) are closed — otherwise the same door is still open.
- **Watch the host closely** for a period after it's back, specifically for the removed
  persistence reappearing, which would mean a foothold was missed.

## Confirm it's over

The incident is closed only when all of these are true:
- Both accounts handled, `analyst` password reset, all three persistence mechanisms gone.
- A hunt for additional footholds came back clean (or the host was reimaged).
- The root-cause gaps — above all, **no MFA** — are fixed, so the same intrusion can't simply
  be repeated tomorrow.

Until that last point is done, containment is temporary: the attacker's *access* is cut, but
the *way in* is still open.
