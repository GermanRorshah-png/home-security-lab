# Incident Analysis — SSH Brute-Force Campaign

**Analyst:** V. Hovta
**Environment:** Personal production VPS (Ubuntu 24.04, internet-facing)
**Data sources:** `fail2ban` status, `/var/log/auth.log`, `/var/log/auth.log.1`
**Type:** Distributed SSH brute-force / credential access attempt

> **Note:** This is a real, live server. Server IP, SSH port, hostname and the
> administrative username are redacted. Attacker IPs are real and publicly
> observable (already blocked).

---

## 1. Summary

The server is continuously targeted by automated SSH brute-force activity from
a large, distributed set of hosts. Over the observed period, fail2ban blocked
**2,700 unique IP addresses** after **3,832 failed authentication attempts**.

No attacker succeeded. The server only accepts SSH key authentication, so
password guessing cannot work even with a correct username. fail2ban removed
repeat offenders automatically.

This document triages the campaign: scope, top sources, the username dictionary
used by the bots, a timeline of the most aggressive attacker, why the server
held, and recommendations.

---

## 2. Scope of the activity

| Metric | Value |
|--------|-------|
| Unique IPs blocked (fail2ban, sshd jail) | 2,700 |
| Total failed authentication attempts | 3,832 |
| Currently active attackers at time of snapshot | 2 |
| Most aggressive single source | 198.50.202.93 (978 attempts) |
| Authentication method accepted by server | SSH public key only (ED25519) |

---

## 3. Top attacking sources

Most failed attempts by source IP (from `auth.log` + `auth.log.1`):

| Attempts | Source IP |
|----------|-----------|
| 978 | 198.50.202.93 |
| 163 | 45.148.10.121 |
| 80  | 173.212.213.201 |
| 42  | 38.12.5.206 |
| 41  | 189.50.142.82 |
| 39  | 167.99.4.252 |
| 37  | 129.121.33.174 |
| 36  | 200.155.66.2 |
| 35  | 103.52.153.215 |
| 35  | 103.248.120.6 |

One source (198.50.202.93) accounts for roughly 6x the volume of the next
highest, indicating a single persistent automated host rather than evenly
distributed traffic. The remainder is spread across many networks
(cloud providers, compromised hosts and residential ranges across multiple
regions), consistent with botnet-style scanning.

---

## 4. Username dictionary observed

The bots cycled through a predictable list of default and service account
names. Top usernames attempted:

| Attempts | Username |
|----------|----------|
| 323 | admin |
| 169 | ubuntu |
| 132 | user |
| 107 | test |
| 89  | ftpuser |
| 61  | debian |
| 54  | oracle |
| 48  | postgres |
| 46  | deploy |
| 45  | dev |
| 35  | git |

Also seen further down the list: `jenkins`, `odoo`, `steam`, `ubnt`,
`AdminGPON` — i.e. default accounts for CI tools, business software, game
servers and networking gear. This is generic opportunistic scanning, not a
targeted attack: the bots try common credentials against anything reachable.

The real administrative account was **never** in the attempted list — it is not
a guessable default, which already defeats this entire class of attack.

---

## 5. Timeline — most aggressive attacker (198.50.202.93)

Source `198.50.202.93` repeatedly attempted to authenticate as **root** at an
almost fixed interval of ~3 minutes 15 seconds:

```
2026-06-14 00:00:36  root  port 42898  Connection closed [preauth]
2026-06-14 00:03:53  root  port 39408  Connection closed [preauth]
2026-06-14 00:07:12  root  port 54242  Connection closed [preauth]
2026-06-14 00:10:22  root  port 33172  Connection closed [preauth]
2026-06-14 00:13:36  root  port 32878  Connection closed [preauth]
...                  ...               (continues at the same cadence)
2026-06-14 01:19:56  root  port 52706  Connection closed [preauth]
```

**Analyst observation:** the regular ~3m15s spacing is deliberate. By keeping
the rate low, the bot tries to stay under simple rate-based detection
thresholds and avoid an immediate ban, while still grinding away over hours.
All attempts target `root` and all are dropped at the pre-authentication stage,
because root login over SSH is disabled and only key authentication is allowed.

---

## 6. Why the server held

The attack volume is high, but impact is zero. Existing controls:

- **SSH key authentication only** — password authentication is disabled, so
  password guessing cannot succeed regardless of username.
- **Root login disabled** — the most-attempted account (root) cannot log in
  over SSH at all.
- **Non-default SSH port** — reduces noise from the most basic scanners.
- **fail2ban (multiple jails)** — automatically bans repeat offenders;
  2,700 IPs blocked from the sshd jail alone.
- **Firewall rules** — limit exposed services.

Legitimate access is confirmed in the logs as successful **publickey**
authentication (ED25519) from the administrator's own host — no passwords
involved.

---

## 7. MITRE ATT&CK mapping

| Technique | ID | Evidence |
|-----------|----|----------|
| Brute Force | T1110 | 3,832 failed attempts across 2,700 IPs |
| Brute Force: Password Guessing | T1110.001 | Repeated root/admin attempts from single sources |
| Valid Accounts (attempted) | T1078 | Dictionary of default/service usernames |

Attack pattern: **distributed credential-access scanning → password guessing
against default accounts → blocked pre-authentication.**

---

## 8. Recommendations

Already in place (validated by this analysis):
- Key-only SSH authentication
- Root login disabled
- Non-default SSH port
- fail2ban active with effective banning

Further hardening to consider:
- Restrict SSH to the WireGuard VPN interface only, so the SSH port is not
  exposed to the public internet at all (removes the entire attack surface).
- Add CrowdSec or an IP reputation blocklist to pre-emptively drop known-bad
  ranges before they reach sshd.
- Forward auth logs to a central location / lightweight SIEM for longer
  retention and trend analysis.

---

*This analysis was produced from live data on my own server as part of my
transition into information security. Currently working towards CompTIA
Network+, Security+ and AZ-500.*
