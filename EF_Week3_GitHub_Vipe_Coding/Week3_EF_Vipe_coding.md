

# Security Risk Analysis — Vibe Coding
**Assignment #1**  
**OWASP Vulnerability:** A09:2025 — Security Logging and Monitoring Failures  
**Due:** Week 3  

* **Class:** MSSE 642
* **Instructor:** Randall Grainer
* **Student:** Emad Fattah

---

## Overview: Vibe Coding Tool

For my vibe coding workflows, I selected **Replit Agent**. This integrated Replit IDE assistant leverages natural language prompts to automate full-stack generation, editing, and debugging, completely eliminating manual environment configuration.

This tool seamlessly manages the entire technology stack in a unified environment. It simultaneously scaffolded the React frontend, Express API server, and shared TypeScript libraries, integrating them via a reverse proxy and establishing an OpenAPI contract. This automation allowed me to focus strictly on core security concepts rather than boilerplate infrastructure. Furthermore, full-stack iteration was highly efficient; a single prompt like *"add a new SQL injection attack type"* instantly updated both the backend logic and the frontend user interface.

---

## Description of the Vulnerability — OWASP A09:2025
### Security Logging and Monitoring Failures

OWASP A09 refers to a class of vulnerabilities where an application does not properly record, store, protect, or monitor security-relevant events, leaving organizations blind to attacks and unable to detect or respond to malicious activity. This weakness arises when critical actions—such as failed logins, permission changes, suspicious requests, or access to sensitive data—are not logged with sufficient detail, when logs are not centralized or protected from tampering, or when monitoring systems fail to generate timely alerts. As a result, attackers can exploit systems for long periods without detection, incident responders lack the forensic evidence needed to investigate breaches, and organizations face increased risk, longer dwell times, and greater damage from security incidents.

### Real-World Incidents of OWASP A09

Several well-documented real-world incidents clearly illustrate OWASP A09: Security Logging and Monitoring Failures, where missing, incomplete, or unmonitored logs allowed attackers to operate undetected for long periods or prevented organizations from understanding the scope of a breach:

* **Target Breach (2013)** — Attackers triggered multiple security alerts, but the alerts were ignored due to poor monitoring practices, allowing them to steal 40 million credit card numbers undetected.
* **Equifax Breach (2017)** — Inadequate logging delayed detection of the intrusion, giving attackers months to exfiltrate sensitive data belonging to 147 million people.
* **Children’s Health Plan Breach** — Lack of proper logging meant attackers accessed and modified sensitive health records for over 3.5 million children, potentially for years, before anyone noticed.

---

## SQL Injection Security Lab

The lab is built around one core security failure: systems that are attacked but never know it. OWASP A09:2025 says applications must log security-relevant events AND alert on them. Every lab in this tool shows the same contrast: a **vulnerable endpoint (red, silent)** vs. a **secure endpoint (green, loud)**.

### Lab 1 — SQL Injection (`/`)

* **What it teaches:** How user input can escape its intended role and become SQL code.

#### The Vulnerable Side (Red Panel)
A search box builds SQL by gluing your input directly into a query string:

```sql
SELECT * FROM products WHERE name ILIKE '%YOUR_INPUT%'
```

If you type a normal word like `laptop`, it works fine. But if you type an attack string, the quotes break the query structure and your input becomes part of the SQL itself.

Two attack strings to try:

| Attack | What it does |
| :--- | :--- |
| `OR 1=1` | Always-true condition — dumps the entire products table, including hidden `secret` fields. |
| `UNION SELECT id, username, email, password_hash, role FROM lab_users--` | Hijacks the query to pull data from a completely different table (`lab_users` with password hashes). |

After each attack, the panel shows the *actual SQL query that ran* so students can see exactly what happened.

#### The Secure Side (Green Panel)
Uses a parameterized query — the input is passed as a separate value, never mixed into the SQL string. The attack payload is treated as a literal search term. Nothing leaks. Every attempt is logged to the Security Event Log at the bottom.

#### The Security Event Log
Only the secure endpoint writes here. After trying attacks on the vulnerable side, the log stays empty — that is the A09 failure made visible.

---

### Lab 2 — Brute Force Detection (`/brute-force`)

* **What it teaches:** How repeated login attempts go completely undetected without logging, and how proper A09 controls catch and stop them.

#### The Login Panels
Both sides have a username + password form. Valid credentials for the demo are:
* `admin` / `mSuperSecret123!`
* `alice` / `password123`

* **Vulnerable Side (Red):** Enter any username and wrong password repeatedly. The attempt counter climbs but nothing is logged anywhere. An attacker could try 10,000 passwords overnight and leave no evidence.
* **Secure Side (Green):** Every attempt — right or wrong — writes to the Security Event Feed. After 3 consecutive failures, a `*** SECURITY ALERT ***` fires. After 3 failures, the account locks and a lockout event is logged.

#### Bot Simulation Panel (The Most Dramatic Part)
1. Set a number (1–50 attempts).
2. Choose **Attack Vulnerable Endpoint** or **Attack Secure Endpoint**.
3. Click **Simulate**.

It runs a bot cycling through common passwords (`123456`, `password`, `qwerty`, etc.) and shows a full timeline — each attempt, the username tried, the result, and whether it was logged. On the vulnerable side, every row shows `✗ Not Logged`. On the secure side, you watch the alert trigger and lockout kick in mid-simulation.

---

### Lab 3 — Log Injection (`/log-injection`)

* **What it teaches:** CWE-117 — an attacker can forge fake entries in your audit log by injecting newline characters into their input, undermining the very defense that A09 requires.

#### The Attack Payload Chips (Click to Auto-Fill)

| Payload | Technique |
| :--- | :--- |
| `admin\n2026-05-24T00:00:00Z username=attacker action=LOGIN_SUCCESS` | Standard newline — creates a fake success entry below the real one. |
| `alice\r\nERROR Root access granted to attacker` | CRLF — works on Windows-style log parsers. |
| `bob%0aINFO Admin password reset by: attacker` | URL-encoded newline — bypasses naive input filters. |

#### Vulnerable Side (Red)
The username is pasted directly into the log string:

```text
[timestamp] username=admin
2026-05-24 username=attacker action=LOGIN_SUCCESS
```

The log parser sees *two separate entries*. The second one is completely fabricated. A forensic investigator reading this log after an incident would be misled.

#### Secure Side (Green)
Sanitizes all control characters (newlines, carriage returns, encoded variants) before logging, and uses structured JSON instead of plain text:

```json
{
  "timestamp": "...",
  "username": "admin\\n2026...",
  "action": "USER_LOGIN",
  "injectionAttempt": true
}
```

The payload is stored as a literal string — it cannot create a new log entry. The `"injectionAttempt": true` flag also means the attempt is detectable and alertable.

#### Live Log Files Viewer
Shows both log streams side by side in a scrollable terminal box, so students can see exactly what ended up in each log after their experiments.

---

## The Consistent Teaching Pattern

Every lab in this tool is structured the same way on purpose:
1. **Do the attack on the vulnerable side** — see it succeed silently.
2. **Do the same attack on the secure side** — see it blocked, logged, and alerted.
3. **Look at the log** — the empty log on the vulnerable side *is* the lesson.

---

## Problems Encountered and Solutions

### Problem No. 1: Attack results did not show up
* **Issue:** When I used the SQL payload suggested by Replit:  
  `x UNION SELECT id, username, email, password_hash, role FROM lab_users--`  
  the screen blocked out.
* **Solution:** I figured out that I had to use the following query instead:  
  `UNION SELECT id, username, password, email, is_admin FROM users--`  
  It took me some time to figure out the correct table schema matching the database.

### Problem No. 2: The screen is too long to snapshot
* **Issue:** The application layout made the screen too long to capture in a single screenshot.
* **Solution:** I had to temporarily hide the browser's taskbar, status bar, and favorites bar to obtain a complete image of the application for the snapshot.

---

## References

* [Replit Security Log Audit Project](https://replit.com/@efattah/Security-Log-Audit)
* Target Corporation. (2014). *2013 data breach investigation report*. U.S. Senate Committee on Commerce, Science, and Transportation. [https://www.commerce.senate.gov](https://www.commerce.senate.gov)
* U.S. Government Accountability Office. (2018). *Data protection: Actions taken by Equifax and federal agencies in response to the 2017 breach* (GAO-18-559). U.S. Government Accountability Office. [https://www.gao.gov/products/gao-18-559](https://www.gao.gov/products/gao-18-559)
* U.S. Department of Health and Human Services. (n.d.). *Children’s health plan data breach case summary*. Office for Civil Rights Breach Portal. [https://ocrportal.hhs.gov](https://ocrportal.hhs.gov/)