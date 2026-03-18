# 🛡️ AI-Powered Windows Security Health Check (2026)

> Combining **Cisco Cybersecurity Essentials** skills with **Machine Learning (scikit-learn)** to produce a practical, SOC-ready security audit tool for Windows 10/11.

---

## ✅ What It Does

| Layer | What's Collected | How It's Analysed |
|---|---|---|
| **Firewall** | Profile state, default actions, logging settings | Rule check — flags disabled profiles instantly |
| **Antivirus / Defender** | Product state, signature age, real-time protection | State code check + age threshold |
| **Processes** | Name, PID, CPU, memory, path, company, start time | Isolation Forest ML + threat-intel rules |
| **Network** | Established TCP connections + remote ports | Known C2/RAT port matching (4444, 1337, 31337…) |
| **Failed Logins** | Event ID 4625 (last 24 h) | Count threshold → brute-force flag |
| **Scheduled Tasks** | Non-disabled tasks + their executable paths | Path-pattern matching for persistence |
| **Startup Items** | HKLM/HKCU Run registry keys | Path-pattern matching for persistence |

---

## 🚀 Quick Start

### Prerequisites
- Windows 10 / 11
- PowerShell 5.1+ (**run as Administrator**)
- Python 3.9+

### Step 1 — Collect system data (PowerShell)

```powershell
# Open PowerShell as Administrator
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\security-check.ps1
```

This creates a timestamped folder `logs_YYYYMMDD_HHMMSS\` containing:

```
logs_20250318_143022/
├── firewall-status.csv
├── antivirus-status.csv
├── defender-status.csv
├── process-logs.csv
├── network-connections.csv
├── failed-logins.csv
├── scheduled-tasks.csv
└── startup-items.csv
```

### Step 2 — Run ML anomaly detection (Python)

```bash
pip install -r requirements.txt
python analyze-logs.py --logdir logs_20250318_143022
```

### Step 3 — Review results

```
logs_20250318_143022/
├── anomalies-processes.csv   ← ML-flagged + rule-hit processes
├── anomalies-network.csv     ← C2/RAT port connections
├── anomalies-startup.csv     ← Suspicious registry startup items
└── security-report.md        ← Human-readable summary with risk score
```

---

## 🧠 How the ML Works

The Python script uses **Isolation Forest**, an unsupervised algorithm that learns the "normal" profile of your running processes and flags statistical outliers.

**Features used:**

| Feature | Why it matters |
|---|---|
| `CPU` | Cryptominers and ransomware spike CPU |
| `WS` (Working Set) | Memory-resident malware inflates this |
| `PM` (Private Memory) | Injected processes show abnormal PM |
| `Handles` | Rootkits and keyloggers open unusual handle counts |
| `Threads` | Shellcode injection changes thread counts |

Features are **StandardScaler-normalised** before fitting so no single feature dominates.

**On top of ML**, threat-intel rules flag:
- Process names matching a known malware/dual-use tool list (mimikatz, netcat, meterpreter, etc.)
- Processes running from `\Users\`, `\Temp\`, `\AppData\` (legitimate system processes don't live there)
- Processes with no digital signature company field (common in hollowed/injected processes)

---

## 📊 Risk Scoring

The final report includes a **0–100 risk score**:

| Score | Severity | Recommended Action |
|---|---|---|
| 0–19 | 🟢 LOW | Review report, no immediate action needed |
| 20–49 | 🟡 MEDIUM | Investigate flagged items within 24 h |
| 50–100 | 🔴 HIGH | Consider isolating machine; escalate to SOC |

Score components:
- Process anomalies: up to 30 pts
- C2 connections: up to 30 pts
- Brute-force logins: 20 pts
- Suspicious startup/tasks: up to 20 pts

---

## 🗂️ Repo Structure

```
ai-windows-security-check-2026/
├── README.md               # This file
├── security-check.ps1      # PowerShell data collector
├── analyze-logs.py         # Python ML + rule engine
├── requirements.txt        # Python dependencies
├── sample-data/            # Example outputs for testing
│   ├── process-logs.csv
│   ├── firewall-status.csv
│   ├── antivirus-status.csv
│   └── anomalies.csv
├── .gitignore
└── LICENSE                 # MIT
```

---

## 🔒 Real-World SOC Notes

1. **False positives are expected** — Isolation Forest flags ~5% of processes by design. Always cross-reference flagged process hashes on [VirusTotal](https://www.virustotal.com).

2. **Run on a schedule** — Use Windows Task Scheduler to run `security-check.ps1` nightly and feed results into your SIEM or a shared drive.

3. **Expand the threat-intel list** — The `SUSPICIOUS_NAMES` set in `analyze-logs.py` can be extended with your organisation's watchlist or IOC feeds (e.g., from MISP, AlienVault OTX).

4. **Event log access** — Failed login collection (Event ID 4625) requires the Security log to be enabled and the script to run as Administrator.

5. **MITRE ATT&CK mapping** — The checks in this tool map to:
   - T1059 — Scheduled task / startup item persistence
   - T1071 — C2 communication on non-standard ports
   - T1110 — Brute force login detection
   - T1055 — Process injection (no-path / no-company flags)

---

## 📦 Dependencies

```
pandas
scikit-learn
numpy
matplotlib    # optional — for future chart exports
```

Install: `pip install -r requirements.txt`

---

## 📄 License

MIT — free to use, modify, and distribute.

---

*Built with 🛡️ by Badger-analyst | Cisco Cybersecurity + Oracle ML Fundamentals | 2026*
