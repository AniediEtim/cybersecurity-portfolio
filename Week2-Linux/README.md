# Week 2 — Linux & SSH Log Analysis

## Overview
Investigated SSH authentication logs on a live Linux machine
to practice real SOC analyst skills — parsing logs, reconstructing
a timeline, identifying suspicious events, and pivoting between
entities.

**Tools Used:** Kali Linux, journalctl, grep, cat, last, who
**Environment:** Local Kali Linux machine

---

## Screenshots

### Full SSH Log Reading
![Full WK2](images/FULL_WK2_redacted.jpg)

### Journalctl — Failed and Invalid Login Detection
![Journalctl](images/JOURNALCTL_redacted.jpg)

### SSH Login Analysis Summary
![SSH Login Analysis](images/SSH%20login%20analysis%20and%20security%20review.png)

### File Permission Hardening
![Permissions](images/PERMISSION.JPG)

---

## What I Did
- Read SSH logs using journalctl and cat /var/log/auth.log
- Filtered accepted and failed logins using grep
- Reconstructed attack timeline from log timestamps
- Identified brute force pattern — multiple failures then success
- Demonstrated chmod permission hardening on sensitive files

---

## Key Findings
- Multiple failed login attempts detected — brute force indicator
- Authentication timeout from 127.0.0.1 — suspicious local connection
- Accepted password login confirmed on port 45994
- File permissions changed from 777 to 600 — security hardening applied

---

## Full Analysis
[View Full SSH Log Analysis](./ssh_log_analysis_week2.md)
