# caldera-adversary-emulation-lab
CTI Lab 7 &amp; 8 — Automated Adversary Emulation using MITRE Caldera v5.3.0 + ELK SIEM | T1033, T1087, T1057 ATT&amp;CK TTPs executed | Detection gap analysis with auditd remediation


# 🎯 Automated Adversary Emulation & Detection Lab


**Tools:** MITRE Caldera v5.3.0 + ELK Stack v8.19.15

---

## 🖥️ Lab Environment

| Node | Machine | IP | Role |
|------|---------|-----|------|
| Attacker / C2 | Kali Linux | 192.168.17.128 | MITRE Caldera :8888 |
| Victim | Ubuntu Linux | 192.168.17.130 | Sandcat Agent + Filebeat |
| SIEM | Debian 13 | 192.168.10.8 | Elasticsearch :9200, Kibana :5601 |

**Operation:** `Initial_Discovery_Test` | Adversary: Discovery | Planner: Atomic

---

## 📋 Lab Phases

### Phase 1 — Caldera Deployment (Kali Linux)
- Cloned MITRE Caldera from GitHub, installed Python dependencies
- Built Magma Vue UI and started server in insecure mode
- Active plugins: sandcat, stockpile, fieldmanual, response, magma

### Phase 2 — Sandcat Agent Deployment (Ubuntu Victim)
- Generated one-liner from Caldera UI → deployed on Ubuntu victim
- Agent `avuvoq` reported **ALIVE** via HTTP beacon to C2 on port 8888
- Agent confirmed **trusted** in Caldera dashboard

### Phase 3 — Operation Execution (MITRE ATT&CK TTPs)

| Time | Status | Technique | Output |
|------|--------|-----------|--------|
| 12:31:15 | ✅ SUCCESS | T1033 — System Owner/User Discovery | `ubuntu` |
| 12:32:01 | ✅ SUCCESS | T1087 — Account Discovery | Local user list from /etc/passwd |
| 12:33:41 | ✅ SUCCESS | T1057 — Process Discovery | Full process list returned |

### Phase 4 — Detection in ELK (SIEM)
- Elasticsearch + Kibana fully operational — 3,786 Ubuntu logs ingested
- KQL queries run for `whoami`, `hostname`, port 8888 beacon traffic
- **Detection gap identified:** process-level events not captured (auditd missing)

---

## 🔍 Detection Gap & Remediation

**Root Cause:** Filebeat was reading `/var/log/syslog` only — does not capture individual process executions.

**Fix:**
```bash
sudo apt install auditd
sudo auditctl -a always,exit -F arch=b64 -S execve -k process_exec
sudo filebeat modules enable auditd
sudo systemctl restart filebeat
```

After fix, KQL query `process.name : "whoami"` would detect T1033/T1087 in Kibana.

---

## 🛠️ Tools Used

- **MITRE Caldera v5.3.0** — C2 framework & adversary emulation
- **Sandcat Agent** — Victim-side implant (HTTP beacon)
- **ELK Stack v8.19.15** — SIEM (Elasticsearch + Logstash + Kibana)
- **Filebeat** — Log shipping from victim to SIEM
- **Kali Linux / Ubuntu / Debian** — Multi-node lab setup

---

## 🔑 Key Findings

- Caldera successfully automated 3 MITRE ATT&CK Discovery techniques
- T1033, T1087, T1057 all executed with confirmed output on victim
- Detection gap (missing auditd) is a **realistic and common SOC finding**
- Documenting + remediating detection gaps = core purple team deliverable

---

## ⚠️ Disclaimer

All activities were performed in a **controlled virtual lab environment** for educational purposes only. No real systems were targeted or compromised.
