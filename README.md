# 🛡️ Email Spoof Checker

**A free, browser-based email security tool built by [Great IT Nordic](https://greatit.se/en/).**

No backend. No login. No data leaves your browser except for DNS lookups via Cloudflare and optional VirusTotal API calls.

---

## Features

### 🌐 Domain DNS Check
Live DNS lookup of all email authentication records for any domain:
- **SPF** — checks whether a record exists, what policy is set (`-all`, `~all`, `+all`, `?all`), and warns if DNS lookup depth approaches the 10-lookup RFC limit
- **DMARC** — checks whether a record exists, what enforcement policy is set (`none`, `quarantine`, `reject`), and whether reporting is configured
- **DKIM** — probes 30 common selectors to find published public keys (`selector1`, `selector2`, `google`, `mailgun`, `ses`, `postmark`, `brevo` and more)
- **MX** — checks whether the domain is configured to receive email
- **BIMI** — checks for a Brand Indicators for Message Identification record (logo in email clients)
- **MTA-STS** — checks whether TLS enforcement for inbound SMTP is configured

Results include an animated score ring (0–100%), a plain-English verdict, and a deliverability risk summary. Every check includes an explanation of what it means and why it matters.

---

### 📧 Header Analysis
Paste raw email headers to detect:
- **From / Return-Path alignment** — mismatches are a classic spoofing indicator
- **Reply-To mismatch** — the most common phishing technique (From looks legitimate, replies go to attacker)
- **Authentication-Results** — parses the SPF/DKIM/DMARC pass/fail written by the receiving server
- **Mail hop count** — high hop counts can indicate deliberate routing to obscure origin
- **Sending client fingerprint** — identifies bulk sending tools and automation

---

### 📋 Bulk Domain Checker
Check up to 50 domains at once — ideal for:
- Supplier and partner email security audits
- Client onboarding assessments
- Internal domain portfolio reviews

Results display in a colour-coded table and can be **exported as CSV** for reporting. Click any domain row to drill into its full check results.

---

### 🕐 History & Change Detection
Every domain check is saved locally in your browser. When you re-check a domain, the tool compares the new result against the previous one and highlights any changes:

- SPF policy changed from `~all` → `-all`
- DMARC moved from `p=none` → `p=quarantine`
- DKIM key added or removed

Each history entry shows a **score sparkline** — a small inline chart tracking the domain's score across the last 10 checks, so you can see remediation progress at a glance.

**Watchlist / starred domains** — star any domain to add it to your watchlist. A "Recheck Starred" button re-runs all watched domains sequentially so you can monitor your key domains in one click.

History supports **search/filter** by domain name.

---

### 📊 DMARC Report Parser *(new in v3.0)*
Paste a DMARC aggregate report (XML) received at your `rua=` address to get a human-readable breakdown:

- Reporter, domain, policy, and date range at a glance
- **Pass rate** — percentage of messages that passed DMARC
- **Source table** — every sending IP with per-source DKIM result, SPF result, disposition, and message count
- Failure callout — highlights unauthorised senders and explains what to investigate

Parsing is done entirely in your browser. No XML data is sent anywhere.

---

### 🔗 Phishing URL Scanner
Paste a suspicious URL from an email and check it against 90+ antivirus and URL reputation engines via the [VirusTotal API](https://virustotal.com).

**Requires a free VirusTotal API key** — [register here](https://www.virustotal.com/gui/join-us) (free tier, no credit card).

- Submits the URL to VirusTotal
- Polls for completed analysis
- Shows detection count, verdict, and per-engine results
- Links directly to the full VirusTotal report

> ℹ️ Your API key is stored only in your browser's localStorage. It is never transmitted to Great IT Nordic's servers.

---

### 🔐 Attachment Hash Checker
Check a suspicious file hash (MD5, SHA-1, or SHA-256) against VirusTotal's database — **no file upload required**. Safe to run from any device.

**How to get a file hash before opening a suspicious attachment:**

```powershell
# PowerShell (Windows)
Get-FileHash .\suspicious.docx | Select Algorithm, Hash

# Or check MD5 specifically
Get-FileHash .\suspicious.docx -Algorithm MD5 | Select Hash
```

Shows file metadata, detection count, first/last seen dates, and per-engine results.

---

### ⚙️ PowerShell Fix Generator
After a domain scan, generates a complete, ready-to-run PowerShell and DNS fix script tailored to exactly what is missing or misconfigured:

- **SPF missing** → generates the exact DNS TXT record to publish
- **SPF `+all`** → shows the exact change needed
- **DKIM not found** → generates Exchange Online `Enable-DkimSigning` command and CNAME record instructions
- **DMARC missing** → generates a three-step DNS record upgrade path (`none` → `quarantine` → `reject`)
- **DMARC `p=none`** → generates the next-step upgrade record
- Always includes a **verification block** with `Resolve-DnsName` commands to confirm changes

Scripts use PowerShell syntax highlighting and can be copied to clipboard in one click.

---

## Learn Panels

The tool includes three built-in education panels:

| Panel | Contents |
|-------|----------|
| 📚 What are SPF, DKIM, DMARC and MX? | Tabbed explanations of each record type, with example DNS records, policy options, and what happens without them |
| 📋 How to extract email headers | Step-by-step instructions for Outlook Desktop, OWA, Gmail, and Apple Mail |
| 🚨 Spotting a spoofed email | 8 common warning signs + a 5-point mental checklist for non-technical users |

---

## Deployment

This is a **single HTML file** with no build step, no dependencies, and no server required.

### GitHub Pages (recommended)

```bash
# 1. Fork or clone this repo
git clone https://github.com/great-it-nordic/email-spoof-checker.git

# 2. Rename the file to index.html if needed
mv email-spoof-checker.html index.html

# 3. Push to GitHub, then:
# Settings → Pages → Deploy from branch: main / (root)
# Live at: https://{username}.github.io/email-spoof-checker
```

### Local use
Just open `email-spoof-checker.html` in any modern browser. No setup needed.

---

## Technical Details

| Component | Technology |
|-----------|------------|
| DNS lookups | [Cloudflare DNS-over-HTTPS](https://cloudflare-dns.com/dns-query) (public, no API key) |
| URL/Hash scanning | [VirusTotal API v3](https://developers.virustotal.com/reference) (free tier key required) |
| History storage | Browser localStorage (local only, no server) |
| Fonts | [DM Sans](https://fonts.google.com/specimen/DM+Sans) + [IBM Plex Mono](https://fonts.google.com/specimen/IBM+Plex+Mono) via Google Fonts |
| Framework | Vanilla HTML/CSS/JS — zero dependencies |

### DNS Lookups
All DNS queries go directly from your browser to `cloudflare-dns.com` using DNS-over-HTTPS with `Accept: application/dns-json`. No email data, headers, or domain names are sent to Great IT Nordic's servers.

### VirusTotal API
URL and hash check requests go directly from your browser to `virustotal.com`. Your API key is stored in localStorage and sent only to VirusTotal — never to any Great IT Nordic endpoint.

### DKIM Selector Probing
DKIM does not have a standard discovery mechanism, so the tool probes 30 common selectors:

`default`, `google`, `mail`, `selector1`, `selector2`, `k1`, `dkim`, `proofpoint`, `mimecast`, `s1`, `s2`, `smtp`, `mandrill`, `sendgrid`, `mailgun`, `ses`, `postmark`, `pm`, `sparkpost`, `zoho`, `mailjet`, `brevo`, `em`, `dk`, `dkimkey`, `key1`, `key2`, `sig1`, `sig2`, `smtp2go`

If DKIM is reported as "not found" but you know it is configured, check with your email provider for the correct selector name and use the custom selector input.

---

## Scoring

Each domain check produces a weighted 0–100 score. Deductions reflect the real-world impact of each missing or misconfigured record:

| Check | Fail deduction | Warn deduction |
|-------|---------------|----------------|
| SPF | −35 pts | −15 pts |
| DMARC | −35 pts | −15 pts |
| DKIM | −20 pts | — |
| MX | −12 pts | — |
| BIMI | −3 pts | — |

| Score | Verdict |
|-------|---------|
| ≥72% | 🛡️ Looks Legitimate |
| 45–71% | ⚠️ Review Carefully |
| <45% | 🚨 High Spoof Risk |

A domain with both SPF and DMARC missing scores 0% — which correctly reflects that it can be freely impersonated, regardless of other checks.

---

## Part of the Great IT Nordic Security Toolset

This tool is built and maintained by **[Great IT Nordic](https://greatit.se/en/)** — a Microsoft 365 IT consultancy based in Sweden.

Related tools:
- 🔧 [M365 PowerShell Toolkit](https://great-it-nordic.github.io/m365-ps-toolkit/) — 223+ Exchange Online, Entra ID, Intune, Teams and SharePoint commands with syntax highlighting and one-click copy
- 📁 [M365 Incident Response Playbooks](https://github.com/great-it-nordic) — structured playbooks with KQL queries and PowerShell scripts for common M365 security incidents

---

## License

MIT — free to use, modify and redistribute. Attribution appreciated.

---

## Contributing

Issues, pull requests and suggestions welcome. If you find a domain where DKIM is miscategorised due to an unusual selector, please open an issue with the selector name so it can be added to the probe list.

---

*Built by [Great IT Nordic](https://github.com/great-it-nordic)*
