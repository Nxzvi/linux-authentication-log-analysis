# 🛡️ Linux Authentication Logs & Blue Team Monitoring

A **hands-on learning portfolio** focused on Linux authentication logs, SSH activity, sudo monitoring, and threat-hunting techniques used by **SOC Analysts / Blue Team professionals**.

This repository demonstrates my understanding of:

* Linux log sources and structures
* Detection of brute-force, credential abuse, and privilege escalation
* Practical log analysis using CLI tools
* Event correlation and SIEM-style detection logic

---

## 📌 Objectives

* Understand where critical Linux authentication logs are stored
* Learn how attackers appear in logs during SSH- and sudo-based attacks
* Practice detection logic used in real SOC environments
* Build threat-hunting mindset using raw logs and correlations

---

## 📂 Key Linux Log Files

| Log File            | Description                                      |
| ------------------- | ------------------------------------------------ |
| `/var/log/auth.log` | Authentication events, SSH, sudo (Debian/Ubuntu) |
| `/var/log/secure`   | Authentication events (RHEL/CentOS)              |
| `/var/log/syslog`   | General system and kernel events                 |
| `/var/log/messages` | Daemon and system messages (RHEL-based)          |
| `/var/log/wtmp`     | Binary log of successful logins and reboots      |
| `/var/log/btmp`     | Binary log of failed login attempts              |
| `/var/log/lastlog`  | Last login details for all users                 |
| `/var/log/sudo.log` | Optional dedicated sudo log                      |

📎 *Note:* Log locations vary by Linux distribution.

---

## 🔐 SSH Authentication Analysis

### ❌ Failed SSH Login

```
Jun 30 14:45:21 server sshd[1183]: Failed password for root from 192.168.0.10 port 54022 ssh2
```

**What it tells us:**

* Possible brute-force or dictionary attack
* Source IP attempting access to privileged account

---

### ✅ Successful SSH Login

```
Jun 30 14:45:24 server sshd[1183]: Accepted password for root from 192.168.0.10 port 54022 ssh2
```

🚩 **Detection Insight:**
A successful login after multiple failures may indicate **credential compromise**.

---

## 🔑 SSH Key-Based Authentication

```
Jun 30 15:12:33 server sshd[1542]: Accepted publickey for analyst from 10.0.1.5 port 49201 ssh2
```

**Blue Team Use Case:**

* Track key-based access
* Detect persistence via unauthorized `authorized_keys`

---

## 🚪 Session Closure Monitoring

```
Jun 30 15:45:10 server sshd[1542]: Received disconnect from 10.0.1.5 port 49201:11: disconnected by user
```

Used to:

* Calculate session duration
* Detect long-running or abandoned sessions

---

## 🧑‍💻 Sudo Activity Monitoring

```
Jun 30 14:47:11 server sudo: joao : TTY=pts/0 ; PWD=/home/joao ; USER=root ; COMMAND=/bin/bash
```

**Why this matters:**

* Indicates privilege escalation
* Root shell access is high risk

---

## 🛠️ Linux Commands for Log Analysis

### SSH Logins

```
grep "Accepted password" /var/log/auth.log
```

### Failed Logins

```
grep "Failed password" /var/log/auth.log
```

### Top Attacking IPs

```
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

### Sudo Usage

```
grep "sudo" /var/log/auth.log
```

### Binary Logs

```
last     # successful logins
lastb    # failed logins
```

---

## 🚨 Anomaly Detection Scenarios

| Pattern                 | Security Indicator          |
| ----------------------- | --------------------------- |
| Multiple failed logins  | Brute-force attack          |
| Login at unusual hours  | Insider threat / compromise |
| Unauthorized sudo use   | Privilege escalation        |
| SSH from foreign IP     | External compromise         |
| Frequent user switching | Session hijacking           |

---

## 🔍 Event Correlation Examples

### Brute-Force Pattern

* Single IP → many failed logins
* Multiple usernames from same IP

### Success After Failures

```
Failed password × N → Accepted password
```

🚩 Flag as **high-risk login**

---

## 🧠 Threat Hunting Techniques

### Hunting Root Shell Access

```
grep 'COMMAND=/bin/bash' /var/log/auth.log | grep 'USER=root'
```

### Lateral Movement Detection

```
grep "Accepted password" /var/log/auth.log | grep -vE '192\.168\.0\.1|10\.0\.0\.1'
```

### Suspicious Sudo Commands

```
grep "sudo" /var/log/auth.log | grep -E 'nc|bash|wget|curl'
```

### User Enumeration

```
grep "Invalid user" /var/log/auth.log
```

---

## 📊 SIEM Integration Example (Elastic)

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "event.module": "ssh" } },
        { "match": { "event.outcome": "failure" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  }
}
```

Used to detect **SSH brute-force attempts in real time**.

---

## 🧪 Honeypot Observation (Cowrie)

```
login attempt [admin:123456] from 103.123.98.12
failed login attempt [admin:qwerty]
```

**Insight:**

* Common credentials
* Automated scanning behavior

---

## 🧠 Key Indicators of SSH Attacks

| Indicator                | Description           |
| ------------------------ | --------------------- |
| Multiple failures        | Brute-force           |
| Invalid users            | Enumeration           |
| Success after failures   | Credential compromise |
| Unknown SSH keys         | Persistence           |
| Suspicious sudo commands | Privilege escalation  |

---

## 📈 Skills Demonstrated

* Linux log analysis
* SOC detection logic
* Threat hunting mindset
* Incident investigation
* SIEM-style correlation

---

## 👤 Author

**Nezvi Hussain**
Cybersecurity Graduate | SOC Analyst (Entry-Level)

🔗 LinkedIn: *(add link)*
📁 More Projects: *(add GitHub profile link)*

---

⭐ If you find this useful, feel free to star the repository!
