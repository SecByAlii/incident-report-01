# 1. Scenario — Engagement Background

> **This is a portfolio exercise.** Coquina Eye Care is a fictional practice, the "breach"
> was simulated by me in a home lab, and no real systems or patient data are involved. The
> scenario is written the way a real incident-response engagement begins, because that's the
> skill this project demonstrates: not just *detecting* an attack, but **investigating one,
> explaining how and why it happened, and making sure it can't happen the same way again.**

## The call

**Monday, 7:48 AM.** The practice manager at **Coquina Eye Care** — a small, 3-provider
eye clinic — calls. Over the weekend, the monitoring they'd set up (a Splunk SIEM watching
their machines) lit up with alerts on one of their Windows computers: a flood of failed
logins overnight, then a successful one, then a string of activity no staff member should
have been generating at 2 AM on a Saturday. By the time anyone looked, the alerts had been
firing for hours.

Nobody touched the machine. They want to know three things, in plain English:

1. **Did someone actually get in — and if so, how?**
2. **What did they do once they were inside?**
3. **Is it over, and how do we stop this from happening again?**

I was brought in to answer those questions.

> *Narrative note: Coquina Eye Care is the same fictional practice from my
> [HIPAA Security Risk Assessment](https://github.com/SecByAlii/hipaa-risk-assessment). That
> assessment flagged specific weaknesses — no multi-factor authentication, weak and shared
> passwords, a flat network. This investigation is what it looks like when gaps like those
> go unaddressed and someone walks through them. The two pieces are two halves of the same
> job: find the risk before the breach, and run the breach down after.*

## The environment

Coquina is a small practice with no in-house IT and no security staff — the setup the
investigation has to assume. In scope for this engagement is the one host that generated the
alerts:

- **`win01`** — a Windows machine on the practice network (the kind of front-desk or
  back-office workstation that, in a real clinic, has more access than it should). It was
  reachable for remote login, which is how the trouble started.
- Its logs were being forwarded in real time to a **Splunk** SIEM, so even actions the
  attacker tried to erase locally were preserved off-box. That forwarded log store is the
  primary evidence source for this investigation.

*(In the lab, `win01` is a single Windows 11 VM standing in for the practice's environment.
It forwards its Security log and Sysmon telemetry to Splunk — the same pipeline documented
in [home-lab-soc](https://github.com/SecByAlii/home-lab-soc).)*

## Scope and objectives

**In scope:** the `win01` host and its forwarded logs, for the weekend window the alerts
cover.

**Objectives, in order:**
1. **Confirm and scope the intrusion** — did a compromise occur, and how far did it reach?
2. **Reconstruct the timeline** — a step-by-step account of what the attacker did and when.
3. **Determine root cause** — how they got in, and the specific weakness that allowed it.
4. **Contain and eradicate** — what to shut down, remove, and reset to end the incident.
5. **Prevent recurrence** — the concrete changes that stop this exact path next time.

**Out of scope:** the rest of the practice network (single-host engagement), legal and
breach-notification decisions (flagged to the practice, not handled here), and attribution
to a specific real-world threat actor.

## Deliverable

A DFIR-style incident report: a one-page executive summary the practice owner can actually
read, backed by the technical findings, the evidence behind each one, and a prioritized set
of recommendations.

## How this report is built (the rest of the repo)

| Phase | Document | What it answers |
|-------|----------|-----------------|
| 1 | Scenario (this file) | Who, what, and the engagement scope |
| 2 | The intrusion | *(reproducible)* exactly how the attack was simulated |
| 3 | Triage | What first indicated a compromise |
| 4 | Timeline | Every attacker action, in order, MITRE ATT&CK-mapped |
| 5 | Forensic artifacts | What the attacker left on the host, with evidence |
| 6 | Findings | How they got in, what they touched, why it worked |
| 7 | Containment & eradication | Ending the incident |
| 8 | Report | Executive summary + recommendations |
