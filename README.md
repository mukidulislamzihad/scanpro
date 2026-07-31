<div align="center">

# 🛡️ ScanPro

**A lightweight, all-in-one network & web application vulnerability scanner — written in pure Python, with a CLI and a cross-platform desktop GUI.**

[![Python](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#desktop-gui)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

Port scanning • Service/banner detection • Automated CVE lookup (NVD) • Web header & TLS auditing • HTML/JSON/PDF reporting • Scan history

Built by **[Mukidul Islam Zihad](https://github.com/mukidulislamzihad)** — [GitHub](https://github.com/mukidulislamzihad) · [LinkedIn](https://www.linkedin.com/in/mukidul-islam-zihad)

</div>

---

## Why ScanPro?

`nmap` is the industry standard for port scanning, and this project doesn't try to replace it. ScanPro exists to solve a different problem: **the manual glue-work around a quick security assessment.**

A typical recon workflow looks like: run `nmap` → eyeball the open ports → run `nikto`/`whatweb` separately for the web layer → cross-reference versions against CVE databases by hand → write up a report for the client. That's 3-4 tools and a lot of copy-pasting for a single target.

ScanPro collapses that into **one command (or one click in the GUI):**

| | ScanPro | Typical multi-tool workflow |
|---|---|---|
| Port scan + banner grab | ✅ built-in | `nmap` |
| CVE lookup on detected versions | ✅ automatic (NVD API) | manual / separate tool |
| Web security header & TLS audit | ✅ built-in | `nikto` / `testssl.sh` |
| Exposed `.git` / `.env` checks | ✅ built-in | manual |
| Multi-target scanning | ✅ comma-separated list | scripted loop |
| Scan history | ✅ built-in, local | manual bookkeeping |
| Ready-to-send HTML / PDF report | ✅ one click | separate reporting tool |
| Install | `pip install -r requirements.txt` | multiple binaries/packages |
| GUI for non-CLI users | ✅ `gui.py` | ❌ |

**Use ScanPro when you want:** a fast, single-file, no-heavy-dependency tool for authorized recon, quick client-facing reports, CTFs, learning how vulnerability scanners work under the hood, or as a beginner-friendly alternative/companion to `nmap` + `nikto`.

**Reach for `nmap` instead when you need:** OS fingerprinting, stealth/evasion scan techniques, IPv6, or the hundreds of NSE scripts built up over 25+ years — ScanPro isn't trying to compete there.

---

## Features

- 🔍 **Port scan + service/banner detection** — no `nmap` binary required, pure Python sockets
- 🌍 **IP resolution everywhere** — every scan type (network, web, subdomain) shows the resolved IP alongside the hostname
- 🧬 **Automatic CVE lookup** against detected service versions via the [NVD API](https://nvd.nist.gov/developers/vulnerabilities)
- 🌐 **Web app checks** — missing security headers, TLS/SSL misconfiguration, certificate expiry, exposed sensitive paths (`.git/config`, `.env`, etc.)
- 🕵️ **Server fingerprint fallback** — if the `Server` header is hidden, ScanPro tries a couple of low-noise heuristics (session cookie names, default error-page wording) to guess the underlying stack
- 🧭 **Best-effort OS fingerprint** (`--os-detect`) — lightweight TTL-based OS family guess (Linux/Windows/other), similar in spirit to `nmap -O` but not definitive
- 🥷 **Stealth/timing mode** (`--stealth`) — randomized port order, jittered delays, and lower concurrency so a scan doesn't look like a single burst to a simple rate-based IDS (timing evasion only, not packet-level SYN stealth)
- 🖥️ **Desktop GUI** (Windows/macOS/Linux, Tkinter — no extra GUI deps) alongside the CLI
- ✅ **Fully opt-in scanning in the GUI** — nothing runs automatically; you tick exactly which scans (network, web, CVE, subdomains, nuclei, OS fingerprint, stealth) you want before clicking Start
- ⏹️ **Stop Scan button** — cancel a running scan mid-way (port scan, subdomain enumeration, or nuclei) and still keep whatever results were collected so far
- 🎯 **Multi-target scanning** — scan several hosts in one run (comma-separated in the GUI)
- 🕘 **Local scan history** — every scan is saved on your machine; revisit and re-export any past result
- ☀️🌙 **Light / dark theme toggle** in the GUI
- 📊 **Live progress bar with percentage**, phase-aware (network → CVE → web)
- 🎨 Color-coded terminal & GUI output, severity-ranked (critical/high/medium/low)
- 📄 **Exportable PDF report** (GUI) / **HTML / JSON / CSV / Markdown** (CLI) — ready to attach to a client deliverable
- 🧭 **Subdomain enumeration** across **three independent sources** — crt.sh + certspotter (both certificate-transparency logs) and hackertarget (passive DNS), merged and deduplicated, no API key needed for any of them. **Every** discovered subdomain is resolved to an IP regardless of how many there are (works on domains with hundreds/thousands of subdomains); a configurable number of the resolved/alive ones then also get a live header/vuln check, not just discovery
- 🩹 **Built-in remediation advisor** — every finding (header issue, TLS issue, CVE, nuclei match) comes with a plain-English "how to fix it" suggestion, fully offline/rule-based
- 🧪 **Optional `nuclei` integration** — if `nuclei` is installed, ScanPro runs it and folds the results into the same report/score
- 🧮 **Overall security score (0-100, A-F grade)** — one number that rolls up every finding across all scan types
- 🐳 **Docker image** for the CLI — run ScanPro without installing Python locally
- ⚡ Multi-threaded scanning, configurable thread count & timeout
- 📦 Minimal dependencies — `requests` + `fpdf2` for PDF export

---

## Table of contents

- [Installation](#installation)
- [Desktop GUI](#desktop-gui)
- [CLI usage](#cli-usage)
- [CLI options](#cli-options)
- [Sample report](#sample-report)
- [Project structure](#project-structure)
- [Roadmap](#roadmap--ideas-for-contributors)
- [Legal notice](#️-legal-notice)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

---

## Installation

```bash
git clone https://github.com/mukidulislamzihad/scanpro.git
cd scanpro
python3 -m venv venv && source venv/bin/activate   # optional but recommended
pip install -r requirements.txt
```

> Requires Python 3.8+. On Debian/Kali/Ubuntu, if you skip the virtual environment
> you may need `pip install -r requirements.txt --break-system-packages`.

---

## Desktop GUI

ScanPro ships with a cross-platform desktop GUI (Windows / macOS / Linux),
built with Tkinter — no extra GUI dependencies needed.

```bash
python3 gui.py
```

Fill in one or more targets (comma-separated), then **tick which scans you actually
want to run** — nothing runs automatically. Options are: Network/port scan, Web app
scan, CVE lookup, Enumerate subdomains, Run nuclei, OS fingerprint, and Stealth mode
(the last two apply on top of the network scan). For subdomain enumeration you can
also set the **Subdomain probe limit** (default `150`) — every discovered subdomain
is still resolved to an IP no matter how many there are; this limit only controls how
many of the *resolved/alive* ones get the slower live header/vuln check.

Click **Start Scan** and watch live color-coded results stream into the log panel
with a real-time progress percentage. Changed your mind mid-scan? Click **Stop Scan**
to cancel it cleanly — whatever was found up to that point is kept. When the scan
finishes (or is stopped), use **Save PDF** to export a report — it defaults to your
**Downloads** folder. Every completed scan is also saved to the **History** tab,
where you can revisit and re-export any past result as PDF. Toggle **☀ Light mode /
🌙 Dark mode** from the top-right of the window to match your preference.

> **Linux users:** if you see `ModuleNotFoundError: No module named 'tkinter'`,
> install the system package first, e.g. `sudo apt install python3-tk`
> (Fedora: `sudo dnf install python3-tkinter`). Tkinter ships with the standard
> Windows and macOS Python installers, so no extra step is needed there.

### Docker (CLI only)

A `Dockerfile` is included for running the CLI without installing Python locally
(the GUI needs a display, so it isn't included in the image):

```bash
docker build -t scanpro .
docker run --rm scanpro example.com --no-network       # web-only scan, prints to terminal
docker run --rm -v "$PWD:/out" scanpro example.com -o /out/report.html   # save report to host
```

### Building a standalone executable (optional)

To distribute the GUI as a single `.exe` / `.app` / Linux binary without requiring
Python to be installed, use [PyInstaller](https://pyinstaller.org/) **on the target OS**
(PyInstaller does not cross-compile — build the Windows `.exe` on Windows, the macOS
`.app` on macOS, etc.):

```bash
pip install pyinstaller
pyinstaller --noconsole --onefile --name ScanPro gui.py
```

The packaged binary will be created under `dist/`.

---

## CLI usage

```bash
# Full scan (network + web) with default port range 1-1024
python3 main.py example.com

# Only web app scan
python3 main.py https://example.com --no-network

# Only network scan on specific ports
python3 main.py 192.168.1.10 --no-web --ports 22,80,443,3306

# Custom port range, more threads, save HTML report
python3 main.py example.com --ports 1-65535 --threads 200 -o report.html

# Save as JSON instead
python3 main.py example.com -o report.json

# Use an NVD API key for higher CVE lookup rate limits
python3 main.py example.com --nvd-api-key YOUR_KEY_HERE

# Also enumerate subdomains (via crt.sh) and check each one for issues
python3 main.py example.com --subdomains --subdomain-limit 50

# Also run nuclei (if installed) against the web target
python3 main.py example.com --nuclei --nuclei-severity critical,high

# Best-effort OS fingerprint (TTL-based) alongside the port scan
python3 main.py example.com --os-detect

# Stealth/timing mode: randomized port order, jittered delays, lower concurrency
python3 main.py example.com --stealth

# Export as CSV or Markdown instead of HTML/JSON
python3 main.py example.com -o report.csv
python3 main.py example.com -o report.md
```

## CLI options

| Flag | Description |
|---|---|
| `--ports` | Port range/list for network scan (default `1-1024`) |
| `--no-network` | Skip the network/port scan |
| `--no-web` | Skip the web application scan |
| `--no-cve` | Skip CVE lookup step |
| `--threads` | Concurrent threads for port scanning (default `100`) |
| `--timeout` | Socket timeout in seconds (default `1.0`) |
| `--os-detect` | Best-effort TTL-based OS fingerprint (like a lightweight `nmap -O` — not definitive) |
| `--stealth` | Timing-based evasion: randomized port order, jittered delays, lower concurrency (not packet-level SYN stealth) |
| `-o, --output` | Save report as `.html`, `.json`, `.csv`, or `.md` |
| `--nvd-api-key` | Optional NVD API key (5 → 50 requests/30s) |
| `--subdomains` | Enumerate subdomains via crt.sh + certspotter + hackertarget and check each for issues |
| `--subdomain-limit` | Max number of resolved/alive subdomains to actively probe with a live header check (default `30`; all discovered subdomains are still resolved to an IP regardless of this limit) |
| `--nuclei` | Run `nuclei` against the target's web app, if installed |
| `--nuclei-severity` | Comma-separated severities to pass to nuclei, e.g. `critical,high` |

Every finding across network, web, subdomain, and nuclei checks now also carries a short **remediation suggestion**, and the report includes an overall **security score (0-100, A-F grade)** rolling everything up into one number.

---

## Sample report

Running a scan produces a color-coded terminal summary and, with `-o report.html`,
a standalone dark-themed HTML report listing open ports, banners, matched CVEs
(with CVSS scores), and web-layer findings ranked by severity — ready to hand to
a client or attach to a pentest writeup.

**Desktop GUI in action** — live scan output, open ports, and CVE matches streaming in real time:

![ScanPro GUI running a scan](screenshots/gui-scan.png)

**Scan summary** — severity-ranked web findings and a one-glance results overview:

![ScanPro scan summary](screenshots/gui-summary.png)

---

## Project structure

```
scanpro/
├── main.py              # CLI entry point
├── gui.py               # Desktop GUI (Tkinter) entry point — "ScanPro"
├── modules/
│   ├── network_scan.py  # Port scanning + banner grabbing
│   ├── cve_lookup.py    # NVD API integration
│   ├── web_scan.py      # HTTP headers, TLS, misconfig checks
│   ├── subdomain_enum.py# Subdomain discovery (crt.sh + certspotter + hackertarget) + per-subdomain checks
│   ├── nuclei_scan.py   # Optional nuclei integration
│   ├── remediation.py   # Rule-based fix suggestions
│   ├── report.py        # Console summary + HTML/CSV/Markdown report
│   └── colors.py        # Terminal color helpers
├── screenshots/         # README screenshots
├── Dockerfile
└── requirements.txt
```

---

## Roadmap / ideas for contributors

- [x] Subdomain enumeration
- [x] Nuclei template integration
- [x] Docker image
- [x] CSV / Markdown export (CLI)
- [x] Overall risk score / security grade
- [x] Rule-based remediation suggestions per finding
- [x] IP resolution shown across network/web/subdomain results
- [x] Best-effort OS fingerprint (TTL-based)
- [x] Timing-based stealth/evasion mode
- [x] Server fingerprint fallback when `Server` header is hidden
- [x] Multi-source subdomain enumeration (crt.sh + certspotter + hackertarget) with full IP resolution regardless of domain size
- [x] Stop/cancel a running scan from the GUI
- [x] Fully opt-in scan selection in the GUI (nothing runs unless selected)
- [ ] Subdomain takeover / dangling DNS detection
- [ ] Authenticated scan mode (session cookie support)
- [ ] Scheduled / periodic scans with diff-against-last-run alerts
- [ ] Webhook/Slack notifications on scan completion
- [ ] REST API mode
- [ ] Per-subdomain screenshots (headless browser)

Contributions welcome — see [Contributing](#contributing).

---

## ⚠️ Legal notice

**Only run this tool against systems you own or have explicit written authorization to test.** Unauthorized scanning of systems is illegal in most jurisdictions. This tool is intended for authorized penetration testing, red team engagements, and security research on your own infrastructure.

---

## Contributing

Issues and pull requests are welcome. If you're picking up something from the
roadmap above, feel free to open an issue first to discuss the approach.

## Author

**Mukidul Islam Zihad**
GitHub: [github.com/mukidulislamzihad](https://github.com/mukidulislamzihad)
LinkedIn: [linkedin.com/in/mukidul-islam-zihad](https://www.linkedin.com/in/mukidul-islam-zihad)

## License

MIT — see [LICENSE](LICENSE).
