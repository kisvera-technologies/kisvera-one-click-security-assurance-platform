# KISVERA Security Engine

Scanner and rule-based Assessment Engine for the **One-Click Security Assurance Platform**. It checks a **verified** web target for common security problems and returns evidence-based findings with CVSS scores.

**Status:** prototype · Built at **KISVERA Technologies** by Ayesha M. Rafiq (cybersecurity engineering)

> **Responsible use:** only scan systems you own or have written permission to test. The demo site in this repo is deliberately vulnerable. Run it locally only.

## How it works

Two separate modules, on purpose: the scanner collects, the assessment engine judges.

1. **Scanner** (`scanner.py`): collects raw evidence (HTTP headers, XSS and SQL-error probes, DNS/mail records, subdomains, TLS, open ports). It never judges.
2. **Assessment Engine** (`assessment.py`): matches evidence to written rules (`rules.json`), computes a CVSS v3.1 score from a fixed vector (`cvss.py`) and sets the severity.

- Same evidence always gives the same score.
- **Severity and confidence are separate.** Doubtful evidence (error page, other host, banner-only CVE match) is kept as `Potential Issue` with lower confidence, never hidden.
- Findings are mapped to OWASP, CIS v8, NIST CSF 2.0, ISO 27001:2022 and SOC 2 as readiness guidance, not certification.

## Checks (23 rules)

| Area | What it does |
|---|---|
| HTTP | CSP, X-Frame-Options, X-Content-Type-Options, HSTS, cookie flags, HTTP to HTTPS redirect, version banners |
| XSS | Reflected input probe, static DOM-XSS heuristic |
| SQL injection | Error-based only, reported as Potential Issue |
| CVE | Banner version to CVE (live NVD API 2.0 or offline sample) |
| DNS / mail | SPF, DMARC, DKIM (common selectors) |
| Subdomains | Common names listed, never scanned |
| TLS | Certificate trust, expiry, TLS 1.0/1.1 |
| Network | Fixed TCP port list, UDP 53 |

## Safety

- Refuses unless the job says `verified: true` (the backend proves ownership).
- Refuses private, loopback and link-local addresses (`allow_private` is for local testing only).
- http/https only, one host, same-host redirects only, read-only GET requests, 4 s timeouts, 200 KB response cap.
- Every request is returned in an `audit` list. Authentication headers are never logged or returned.
- **Not done on purpose:** stored XSS, blind/time-based SQL injection, exploitation, brute force and denial-of-service tests.

## Quick start

Pure Python 3 standard library, nothing to install.

```bash
cd engine
python3 tests/run_tests.py                       # 74 checks, local test servers only
python3 run_scan.py https://YOUR-OWN-SITE --verified
```

Scan the local demo site:

```bash
python3 demo/weak_site.py                        # http://127.0.0.1:8081
python3 engine/run_scan.py http://127.0.0.1:8081 --verified --allow-private
```

Use from Python:

```python
from kisvera_engine import run_job

result = run_job({
    "module_id": "kisvera.engine",
    "schema_version": "1.0",
    "target": {"id": "t1", "url": "https://example.com", "verified": True},
})
# result["execution_status"]: completed | rejected | failed
# result["results"]: list of findings
```

## Project structure

```
engine/
  kisvera_engine/   scanner.py, assessment.py, cvss.py, nvd.py, rules.json
  tests/            run_tests.py (74 checks)
  run_scan.py       command-line scan
  flask_glue.py     example of calling the engine from a web backend
demo/               weak_site.py: deliberately weak LOCAL practice site
tools/              smoke_test.py: end-to-end check (needs the full platform)
```

## Known limits

- Heuristic results (DOM XSS, SQL error, banner CVE, missing DKIM) stay Potential Issues until a person confirms them.
- The offline CVE file is a small sample. Live NVD lookup needs internet and should be tested once before a demo.
- The probe crawls only the start page's links (max 5 pages x 3 parameters).

## Team

KISVERA Technologies internship team: Muhammad Aazmeer Khan (front end), Abdul Nafay (backend), Ayesha M. Rafiq (cybersecurity engineering).
