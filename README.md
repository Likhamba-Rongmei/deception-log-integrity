# Deception & Log Integrity — A Two-Layer Security Toolkit

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=flat-square&logo=python)
![Flask](https://img.shields.io/badge/Flask-3.1.3-black?style=flat-square&logo=flask)
![Security](https://img.shields.io/badge/Focus-Cybersecurity-red?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## About the Project

Catching an attacker is only half the job. If they can quietly edit or delete the logs afterwards, the evidence is gone.

I built this project to explore both halves of that problem in Python:

1. **A honeypot** that lures attackers into a fake admin portal and records everything they do.
2. **A tamper-evident logger** that chains every log entry with SHA-256 hashes, so any later edit, deletion or reordering is detected.

| Module | What it does | Core idea | Stack |
|--------|--------------|-----------|-------|
| [`honeypot/`](honeypot/) | Fake corporate login portal that logs visits, credential attempts and brute-force behaviour | Deception-based detection | Python, Flask, HTML/CSS |
| [`tamper-evident-logging/`](tamper-evident-logging/) | Append-only log where each entry is linked to the previous one by hash | Cryptographic hash chaining | Python, hashlib, JSON |

---

## How the Pieces Fit Together

```
Attacker discovers the fake portal         (honeypot)
              ↓
IP, credentials and browser info are logged
              ↓
Log entries are hash-chained                (tamper-evident logging)
              ↓
Attacker tries to cover their tracks
              ↓
Chain verification fails → tampering is pinpointed, evidence preserved
```

- **Layer 1: Detection.** The honeypot has no legitimate users, so any interaction with it is a signal.
- **Layer 2: Integrity.** The hash chain makes the record of that interaction trustworthy after the fact.

> The two modules currently run independently. Piping honeypot alerts straight into the hash-chained logger is next on the roadmap (see below).

---

## Screenshots

### Fake Login Portal
![Login portal](assets/login.png)

### Alert Dashboard
![Dashboard](assets/dashboard.png)

---

## Demo

A walkthrough video and extra screenshots are available here:

[Watch the demo](https://drive.google.com/drive/folders/12l2ERhBdpg-XuLBIeMvKV9ONW1TMgr_n?usp=drive_link)

---

## Getting Started

### Prerequisites

```bash
python --version    # Python 3.9+
pip install flask   # only external dependency
```

### Run the honeypot

```bash
cd honeypot
python app.py
```

| URL | What it shows |
|-----|---------------|
| `http://127.0.0.1:5000/login` | The fake login portal (the trap) |
| `http://127.0.0.1:5000/honeypot-dashboard` | Live alert dashboard (the defender's view) |

Alerts are written to `alerts.log` (human-readable) and `alerts.json` (machine-readable). After 5 login attempts from the same IP, a `BRUTE_FORCE_DETECTED` event is raised.

### Run the tamper-evident logger

```bash
cd tamper-evident-logging
python logger.py
```

On first run it seeds a few sample entries and opens an interactive menu where you can:

- add log entries and view the chain
- verify integrity
- **simulate attacks** (modify, delete or reorder entries) and watch the verifier catch them and report the first compromised entry

---

## Project Structure

```
.
├── assets/
│   ├── dashboard.png
│   └── login.png
│
├── honeypot/
│   ├── templates/
│   │   └── login.html          # Fake "SecureNet" admin portal UI
│   ├── app.py                  # Flask server: routing, alerting, brute-force detection
│   └── README.md               # Design notes and limitations
│
├── tamper-evident-logging/
│   ├── logger.py               # Hash-chained logger, verifier and attack simulator
│   └── README.md               # Design notes and limitations
│
└── README.md
```

> Log files (`alerts.log`, `alerts.json`, `logs.json`) are generated at runtime and are not committed.

---

## What I Learned

- **Deception works because of asymmetry.** A real user never touches the decoy, so you get almost no false positives.
- **Believability matters.** A convincing UI, with a live clock, session IDs and a TLS badge, gets far more interaction than a bare form.
- **Hash chains detect tampering but don't prevent it.** An attacker with full write access can recompute the whole chain, so real systems add external anchoring, signatures or WORM storage.
- **Detection and integrity are separate problems.** Solving one without the other leaves a gap.

---

## Roadmap

- [ ] Feed honeypot alerts directly into the hash-chained logger
- [x] Escape attacker-supplied input on the dashboard (prevents stored XSS against the defender)
- [ ] Protect the dashboard with an auth token or IP allow-list
- [ ] Time-windowed brute-force detection instead of a lifetime count
- [ ] Sign log entries and periodically anchor the latest hash externally
- [ ] IP geolocation and email/webhook alerts
- [ ] Containerise and deploy to a cloud VM to capture real-world traffic

---

## Disclaimer

This project is for educational and defensive research purposes. Deploy honeypots only on infrastructure you own or are authorised to monitor, and handle any captured credentials responsibly.
