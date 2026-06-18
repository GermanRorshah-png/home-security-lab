# Home Security Lab

Personal VPS used as a hands-on lab for practical infrastructure security:
hardening, intrusion prevention, VPN and automated monitoring of a live
internet-facing server.

> **Note:** All sensitive values (server IP, SSH port, usernames, keys) are
> redacted. Attacker IPs shown in logs are real and publicly observable.

---

## Overview

This server runs real production services and is continuously exposed to the
internet, which means it receives constant automated attacks (brute-force SSH,
web scanning). I use it to practise detection, prevention and response in a
real environment rather than a simulation.

## Stack

| Area | Tools |
|------|-------|
| OS | Ubuntu 24.04 (Linux administration) |
| Web | nginx, HTTPS (Let's Encrypt) |
| VPN | WireGuard |
| Intrusion prevention | fail2ban (multiple jails) |
| Hardening | SSH key-only auth, non-default port, firewall rules |
| Monitoring | Python bot with real-time Telegram alerts |
| Scripting | Python, bash |

## What this lab demonstrates

- **SSH hardening** — key-based authentication only, root login disabled,
  non-standard port, firewall rules limiting exposure.
- **Intrusion prevention with fail2ban** — multiple jails covering SSH and web
  services. Over 2000 malicious IPs blocked to date.
- **WireGuard VPN** — encrypted remote access to the server.
- **Automated monitoring** — a Python bot watches logs and sends real-time
  alerts to Telegram (failed logins, bans, service events).
- **Log analysis** — reviewing auth and fail2ban logs to understand attacker
  behaviour (mostly automated SSH brute-force from distributed IPs).

## Mapping to MITRE ATT&CK

The most common activity observed against this server:

| Technique | ID | Evidence |
|-----------|----|----------|
| Brute Force | T1110 | Repeated failed SSH logins from many source IPs, blocked by fail2ban |

## Planned next steps

- Document a full triage of one real brute-force incident (timeline, source IPs, response).
- Add screenshots of the monitoring dashboard and sample alerts.
- Expand detection notes and fail2ban jail configuration (sanitised).

---

*This repository is part of my transition into information security.
Currently working towards CompTIA Network+, Security+ and AZ-500.*
