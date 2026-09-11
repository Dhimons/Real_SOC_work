# Cloudora Alert Triage Shift

A simulated SOC analyst triage exercise: working a 12-alert overnight queue for a fictional B2B HR software company, end-to-end in ServiceNow.

> **Note:** Cloudora is a fictional training client. All data, IPs, domains, and detections in this project are synthetic (part of the [MyFirstHack](https://myfirstcyberjob.com) community training resource — *Alert Triage Shift* exercise). Nothing here is real client data.

## The scenario

Cloudora is a 150-person B2B HR software company in London. In the month before this shift, they had two real incidents:

- **Executive account takeover** — a password-spray attack from three flagged IPs compromised the CEO's account and one other, with a rogue MFA device registered for persistence.
- **Payroll phishing campaign** — a lookalike-domain phishing run hit all 40 staff mailboxes; two employees submitted credentials.

Both were caught late — by luck and by users reporting things — which is why the vCISO instituted a formal triage process. This project is the first shift run under that process: **twelve alerts, arrived overnight, of mixed type and mixed quality**, worked top to bottom.

## Shift handover

Every real shift starts with a handover. Here's the one this queue was worked against:

| | |
|---|---|
| **Change window** | CHG-2088: backup platform credential rotation, 31 Aug 23:00 – 1 Sep 02:30 UTC. Brief auth failures expected. |
| **Scheduled scans** | Authorised vulnerability scanner `LDN-SCAN-01` (`203.0.113.44`) runs a full sweep Monday 22:00 – Tuesday 03:00 UTC, all offices. |
| **Scheduled automation** | HR's annual records retention cleanup (`HR-RetentionFlow`), running under `tara.kemp`'s account in the early hours, deleting expired records per the 7-year retention policy. |
| **Helpdesk** | Shared queue with the SOC; referenced by HD number. Staff are told to contact helpdesk for new devices, lockouts, and travel. |
| **Incident history** | IOCs from both prior incidents remain blocked at Conditional Access and the perimeter: 3 flagged IPs from the account takeover, 3 phishing relay IPs, and 1 flagged sign-in host from the Amsterdam compromise. Watchlist alerts fire on any activity from these. |
| **Escalation SOP** | Escalate to the vCISO immediately when: confirmed malicious activity spans more than one host/site; an executive account shows confirmed (not blocked) activity; confirmed access to client data; or the alert can't be bounded with available data. |
| **Severity policy** | Reported severity is just the detection rule's opinion — every alert gets re-rated using the severity matrix below. |

## Methodology

Every alert gets one of five verdicts:

- **True positive** — malicious activity, confirmed by evidence.
- **False positive** — the rule fired on explained, authorised activity (maintenance, a scanner, user error, legitimate travel).
- **Duplicate** — the same underlying event as another ticket, arrived by a different path.
- **Escalate** — real, or too big to judge at triage level. Escalation is a verdict, not a failure.
- **Insufficient data** — can't yet defend any other verdict. Name what's missing and how to get it.

**Severity = impact × confidence**, not the number the rule shipped with:

| | Confirmed | Probable | Possible |
|---|---|---|---|
| **High impact** (exec account, sensitive data, multiple hosts/sites) | Sev 1 | Sev 2 | Sev 3 |
| **Medium impact** (one standard user/host) | Sev 2 | Sev 3 | Sev 4 |
| **Low impact** (blocked, failed, or no effect) | Sev 4 / Info | Info | Info |

Every verdict is written as one paragraph with seven parts — **alert, hypothesis, evidence checked, verdict, severity, action, justification** — structured so another analyst could read it cold and retrace the reasoning.

## Repo structure

```
cloudora-soc-triage/
├── README.md
├── tickets/
│   ├── CLD-0101-svc-backup-failed-signins.md
│   ├── CLD-0102-malware-quarantine-failure.md
│   ├── CLD-0103-internal-port-scan.md
│   ├── CLD-0104-mfa-failures-then-success.md
│   ├── CLD-0105-reported-payslip-email.md
│   ├── CLD-0106-defender-daily-digest.md
│   ├── CLD-0107-mass-file-deletion.md
│   ├── CLD-0108-first-time-signin-properties.md
│   ├── CLD-0109-multi-host-beaconing.md
│   ├── CLD-0110-watchlist-ip-blocked.md
│   ├── CLD-0111-reported-invoice-email.md
│   └── CLD-0112-impossible-travel.md
├── end-of-shift-summary.md
└── tuning-recommendations.md
```

## Shift outcome

## Detailed Incident Triage & Resolutions

### CLD-0101 — Multiple Failed Sign-ins: svc-backup@cloudora.io

* **Alert ID:** CLD-0101
* **Fired (UTC):** 2026-08-31 23:41:07 UTC
* **Source System:** Identity monitoring (Entra ID)
* **Reported Severity:** High
* **Title:** Multiple failed sign-ins: svc-backup@cloudora.io
* **Affected User:** `svc-backup@cloudora.io`
* **Affected Host:** `LDN-SRV-BK1`
* **Detail:** Rule 'Repeated authentication failures (service account)' fired. 18 failed sign-ins (error 50126, invalid credentials) for `svc-backup@cloudora.io` between 23:41 and 00:03 UTC, source `203.0.113.20` (`LDN-SRV-BK1`, internal backup server), followed by a successful sign-in at 00:14 UTC from the same host.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (233)" src="https://github.com/user-attachments/assets/45d95939-7edf-49b2-88e8-0de4045da5b4" />


---

### CLD-0102 — Malware Detected, Quarantine Failed: LDN-WS-117

* **Alert ID:** CLD-0102
* **Fired (UTC):** 2026-09-01 02:45:33 UTC
* **Source System:** Endpoint protection (Defender AV)
* **Reported Severity:** High
* **Title:** Malware detected, quarantine failed: LDN-WS-117
* **Affected User:** `liam.doyle@cloudora.io`
* **Affected Host:** `LDN-WS-117`
* **Detail:** Rule 'Active threat not remediated' fired. Detection `Trojan:Win32/Stealgen.B` (credential stealer) on `LDN-WS-117` (assigned user liam.doyle). Four detection events between 02:13 and 02:41 UTC; quarantine FAILED each time and the file reappears at `C:\Users\liam.doyle\AppData\Roaming\WinSyncSvc\wsync.exe`.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (234)" src="https://github.com/user-attachments/assets/6cb4b5f0-404f-4725-bea0-3e0efe25e696" />

---

### CLD-0103 — Internal Port Scan Detected from 203.0.113.44

* **Alert ID:** CLD-0103
* **Fired (UTC):** 2026-09-01 02:52:10 UTC
* **Source System:** Perimeter firewall / IDS
* **Reported Severity:** Medium
* **Title:** Internal port scan detected from 203.0.113.44
* **Affected Host:** `LDN-SCAN-01`
* **Detail:** Rule 'TCP port sweep' fired. Host `203.0.113.44` (`LDN-SCAN-01`) probed 4,000+ TCP ports across 180 internal hosts between 22:10 and 02:5x UTC. Traffic pattern: sequential SYN probes, no data transfer. No other alerts from this source.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (235)" src="https://github.com/user-attachments/assets/aa3dea41-343d-4708-8f0b-97135923f5bf" />


---

### CLD-0104 — Repeated MFA Failures Then Success: nina.cole@cloudora.io

* **Alert ID:** CLD-0104
* **Fired (UTC):** 2026-09-01 06:55:26 UTC
* **Source System:** Identity monitoring (Entra ID)
* **Reported Severity:** Medium
* **Title:** Repeated MFA failures then success: nina.cole@cloudora.io
* **Affected User:** `nina.cole@cloudora.io`
* **Detail:** Rule 'MFA failures followed by success' fired. Five MFA denials (error 500121) for `nina.cole@cloudora.io` between 06:31 and 06:47 UTC, then a successful sign-in at 06:52 UTC. All events from `198.51.100.22` (Manchester office), Windows 10, Chrome. A new authenticator device was registered at 06:50 UTC. Related: helpdesk ticket `HD-4471` raised at 07:05 UTC.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (236)" src="https://github.com/user-attachments/assets/7cc56ca3-fb1d-445f-9420-049c3159faad" />


---

### CLD-0105 — User-Reported Suspicious Email: 'Your August payslip is ready'

* **Alert ID:** CLD-0105
* **Fired (UTC):** 2026-09-01 07:58:49 UTC
* **Source System:** User-reported email (`soc@cloudora.io`)
* **Reported Severity:** Medium
* **Title:** User-reported suspicious email: 'Your August payslip is ready'
* **Affected User:** `james.holt@cloudora.io`
* **Detail:** `james.holt@cloudora.io` reported an email received at 07:45 UTC, sender shown as `payroll@cloudora.io`, subject 'Your August payslip is ready', containing a link to a payslip portal. Reporter note: "After last week I am not clicking anything from payroll. Please check." Per post-CLD-0002 policy every user report is triaged.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (237)" src="https://github.com/user-attachments/assets/b95afbd8-071e-4903-a017-245337f82d9f" />


---

### CLD-0106 — Daily Summary: 1 Device with Active Unresolved Threats

* **Alert ID:** CLD-0106
* **Fired (UTC):** 2026-09-01 08:05:00 UTC
* **Source System:** Defender daily digest connector
* **Reported Severity:** Medium
* **Title:** Daily summary: 1 device with active unresolved threats
* **Affected User:** `liam.doyle@cloudora.io`
* **Affected Host:** `LDN-WS-117`
* **Detail:** Automated ticket from the Defender daily email digest (08:00 UTC summary): 1 device with active unresolved threats in the last 24 hours: `LDN-WS-117`, detection `Trojan:Win32/Stealgen.B`, remediation incomplete.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (238)" src="https://github.com/user-attachments/assets/410dd962-8d9e-42ed-8582-f9ac58fc7ad7" />


---

### CLD-0107 — Mass File Deletion: 1,240 Files Removed from SharePoint

* **Alert ID:** CLD-0107
* **Fired (UTC):** 2026-09-01 08:12:41 UTC
* **Source System:** M365 activity alerting
* **Reported Severity:** High
* **Title:** Mass file deletion: 1,240 files removed from SharePoint
* **Affected User:** `tara.kemp@cloudora.io`
* **Detail:** Rule 'Mass deletion by a single user' fired (cloud activity alerts can lag the activity by several hours). 1,240 files deleted from SharePoint site 'HR-Records-Archive', folder 'Leavers-2019', between 03:00 and 03:20 UTC under the account `tara.kemp@cloudora.io`. Activity detail records the initiating process as 'HR-RetentionFlow' (scheduled automation).

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (239)" src="https://github.com/user-attachments/assets/76222cb3-4097-4f57-bef2-cb5903eceaba" />

---

### CLD-0108 — First-Time Sign-in Properties: gwen.muir@cloudora.io

* **Alert ID:** CLD-0108
* **Fired (UTC):** 2026-09-01 08:17:12 UTC
* **Source System:** Identity monitoring (Entra ID)
* **Reported Severity:** Low
* **Title:** First-time sign-in properties: gwen.muir@cloudora.io
* **Affected User:** `gwen.muir@cloudora.io`
* **Detail:** Rule 'Unfamiliar sign-in properties' fired (risk detections are batch-evaluated and can lag). One successful sign-in for `gwen.muir@cloudora.io` at 03:54 UTC from `198.18.163.41` (geolocates to London, United Kingdom; residential provider), client app IMAP4 (legacy protocol), Windows 10, Firefox. No failed attempts preceded it.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (240)" src="https://github.com/user-attachments/assets/a379b613-d8b7-4c10-ba9d-0d0ca22c8886" />

---

### CLD-0109 — Repeated Outbound Connections to Rare External Host (3 Hosts)

* **Alert ID:** CLD-0109
* **Fired (UTC):** 2026-09-01 08:22:55 UTC
* **Source System:** Perimeter firewall / IDS
* **Reported Severity:** Medium
* **Title:** Repeated outbound connections to rare external host (3 hosts)
* **Affected Users:** `aria.reid@cloudora.io`; `finn.gale@cloudora.io`; `dean.page@cloudora.io`
* **Affected Hosts:** `LDN-WS104`; `MAN-LT-033`; `AUS-WS-208`
* **Detail:** Rule 'Periodic outbound connections to uncategorised destination' fired. Since approximately 07:40 UTC, three internal hosts: `LDN-WS104` (`aria.reid`), `MAN-LT-033` (`finn.gale`) and `AUS-WS-208` (`dean.page`), have each made an outbound HTTPS connection to `198.18.207.66` roughly every 60 seconds. Destination is uncategorised by the firewall threat feed and its domain registration is 9 days old. Hosts are in three different offices (London, Manchester, Austin).

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (241)" src="https://github.com/user-attachments/assets/c8871518-c0ba-411b-a53f-949e5e94c605" />


---

### CLD-0110 — Blocked Sign-in Attempts from Watchlist IP: daniel.reeve@cloudora.io

* **Alert ID:** CLD-0110
* **Fired (UTC):** 2026-09-01 08:26:30 UTC
* **Source System:** Identity monitoring (Conditional Access)
* **Reported Severity:** Low
* **Title:** Blocked sign-in attempts from watchlist IP: daniel.reeve@cloudora.io
* **Affected User:** `daniel.reeve@cloudora.io`
* **Detail:** Scheduled analytic 'Watchlist IP activity' fired (runs every few hours). Three sign-in attempts for `daniel.reeve@cloudora.io` between 05:09 and 05:11 UTC from `102.89.44.17` (Lagos, Nigeria), all returned error 53003, blocked by Conditional Access policy. This IP is on the tenant block list: it is attacker infrastructure from incident CLD-0001 (August executive account takeover).
**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (242)" src="https://github.com/user-attachments/assets/4fbc9a93-5411-426b-8d7e-56daae859847" />


---

### CLD-0111 — User-Reported Suspicious Email: 'Overdue invoice INV-2214'

* **Alert ID:** CLD-0111
* **Fired (UTC):** 2026-09-01 08:31:18 UTC
* **Source System:** User-reported email (`soc@cloudora.io`)
* **Reported Severity:** Medium
* **Title:** User-reported suspicious email: 'Overdue invoice INV-2214'
* **Affected User:** `lucas.ford@cloudora.io`
* **Detail:** `lucas.ford@cloudora.io` (accounts payable) reported an email received at 08:19 UTC from `billing@meridianoffice.example` ('Meridian Office Supplies Ltd'), subject 'Overdue invoice INV-2214, second reminder', with attachment `INV-2214.pdf`. Reporter note: "Not sure we use this supplier. Have not opened the attachment. Or I do not think I did."

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (243)" src="https://github.com/user-attachments/assets/ae29d606-ae33-4eed-8bfe-f595175b0471" />


---

### CLD-0112 — Impossible Travel: omar.farah@cloudora.io (Dubai then London)

* **Alert ID:** CLD-0112
* **Fired (UTC):** 2026-09-01 08:44:03 UTC
* **Source System:** Identity monitoring (Entra ID)
* **Reported Severity:** Medium
* **Title:** Impossible travel: omar.farah@cloudora.io (Dubai then London)
* **Affected User:** `omar.farah@cloudora.io`
* **Detail:** Rule 'Atypical travel' fired (this detection is ML-based and lags by design). `omar.farah@cloudora.io` signed in from Dubai, United Arab Emirates at 05:58 UTC and from London, United Kingdom (`203.0.113.10`) at 06:40 UTC, 42 minutes apart. The two locations are roughly a seven hour flight apart.

**ServiceNow Resolution & Proof:**
<img width="1920" height="1080" alt="Screenshot (244)" src="https://github.com/user-attachments/assets/3d194e5c-dc42-49e6-a083-aa8cf687e174" />


## What this demonstrates

I ran an alert triage queue in ServiceNow during a simulated client engagement: twelve mixed alerts, worked to a written verdict standard where every call names its evidence, its severity re-rating, and its justification — the same discipline a real SOC queue runs on.
