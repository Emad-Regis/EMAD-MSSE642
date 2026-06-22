
# Vibe Coding Assignment #3

**OWASP A05:2025 — SQL Injection**  
**Student:** EMAD FATTAH  
**Class:** MSSE 642 – Software Assurance  
**Instructor:** Randall Granier  
**Date:** Jun/2026  

---

## Introduction

A02:2025 – Security Misconfiguration represents one of the most critical and widespread web application security risks today [2]. It covers flaws arising from improperly hardened application stacks, poorly configured cloud services, and default setup parameters [2]. In the current OWASP Top 10:2025 framework, this category surged from #5 to #2, driven by the explosive adoption of modern cloud-native architectures, containerization, and Infrastructure as Code (IaC) [3, 5]. According to data verified by [Qualys Insights](https://blog.qualys.com/qualys-insights/2026/06/15/what-changed-in-owasp-top-10-2025-and-recommendations-for-each-category), an alarming 100% of tested applications showed some form of misconfiguration, resulting in an average incidence rate of 3.00% [3].

---

## Why A02:2025 is Highly Critical ?

Modern software deployment relies heavily on external configurations rather than hardcoded code logic [2]. The primary areas of risk include:

*   **Insecure Defaults:** Deploying application frameworks, content management systems (CMS), or network systems with original factory passwords, default accounts, or active example configurations [2].
*   **Overly Permissive Cloud Storage:** Leaving cloud buckets (e.g., AWS S3, Azure Blobs) or database services publicly accessible without proper authentication token requirements.
*   **Verbose Error Messages:** Applications that expose extensive debugging details, full stack traces, or environment secrets to end-users [2]. Attackers leverage this information to map out infrastructure layouts.
*   **Unnecessary Feature Exposure:** Keeping debugging modes, raw test endpoints, or obsolete communication ports (like Telnet or FTP) actively running within production servers [2].
*   **Missing Security Headers:** Failing to implement baseline server protections such as `Content-Security-Policy` (CSP) or HTTP Strict Transport Security (HSTS).

---

## Recent Attacks Driven by Security Misconfigurations

Attackers rely on automated scanning tools to locate misconfigured platforms across the internet. Recent major global campaigns highlighted by security researchers emphasize this threat:

### 1. The Salesforce Component Campaign (Late 2025)
A coordinated campaign targeted organizations utilizing misconfigured Salesforce communities and cloud applications [4]. Due to open permissions and loose access configurations on public-facing components, attackers extracted sensitive customer data from major technology firms [4]. Documented victims noted by [Infosecurity Magazine](https://www.infosecurity-magazine.com/news-features/top-10-cyberattacks-of-2025/) included BeyondTrust, Cloudflare, Qualys, and Zscaler [4]. 

### 2. Apache Camel Header Exploits (Early 2025)
Threat actors targeted legacy integration pipelines utilizing Apache Camel deployments [6]. Due to unhardened default parsing rules—tracked via CVE-2025-27636—attackers injected arbitrary, malicious headers directly into backend microservices [6]. This bypassed perimeter firewalls and enabled secondary remote exploitation ([Akamai Blog](https://www.akamai.com/blog/security-research/march-apache-camel-vulnerability-detections-and-mitigations)) [6].

### 3. Continuous Cloud Bucket Data Leaks (Ongoing)
Throughout the retail, e-commerce, and SaaS sectors, poorly managed Terraform files and cloud infrastructure scripts have left entire data stores publicly accessible. These architectural oversights have exposed millions of consumer records to threat actors without requiring a single line of malicious code injection.

---

## References

*   efattah. (n.d.). *SQL-injection-defense* [Source code]. Replit. https://replit.com/@efattah/SQL-Injection-Defense [1].
*   CyberSenpai. (2024, May 15). *Security misconfiguration: The hidden risk in web applications*. MeetCyber. https://meetcyber.net/security-misconfiguration-the-hidden-risk-in-web-applications-d5d235812dc4 [2].
*   Dandage, S. (2026, June 16). *What changed in OWASP Top 10 2025 and recommendations for each category*. Qualys Blog. https://blog.qualys.com/qualys-insights/2026/06/15/what-changed-in-owasp-top-10-2025-and-recommendations-for-each-category [3].
*   Author. (2025, December 16). *Top 10 cyber-attacks of 2025*. Infosecurity Magazine. https://www.infosecurity-magazine.com/news-features/top-10-cyberattacks-of-2025/ [4].
*   Heringer, N. (2025, December 3). *Some welcome changes: Dissecting the OWASP Top 10 2025*. Niklas Heringer. https://niklas-heringer.com/security-news/owasp-top-10-2025/ [5].
*   Akamai Security Research. (2025, March 11). *Detecting and mitigating the Apache Camel vulnerabilities*. Akamai Blog. https://www.akamai.com/blog/security-research/march-apache-camel-vulnerability-detections-and-mitigations [6].

---

## Overview (Vibe Coding Tool)

I choose to use Replit to develop my A02:2025 educational tool because its cloud-based environment completely eliminates complex local setups, allowing me to build, test, and instantly share an interactive application through a single browser link [1]. By leveraging Replit's built-in AI capabilities, I can rapidly scaffold real-world security misconfiguration scenarios—such as verbose error pages, unhardened HTTP headers, and default credentials—while ensuring my users get an identical, isolated sandbox to safely practice exploit mitigation [1]. Ultimately, I select this platform because it streamlines my entire development workflow and provides a zero-config, highly accessible learning space for my target audience [1].

---

## 2. Description of the Program

The Program is an interactive browser-based sandbox that teaches *OWASP A02:2025 — Security Misconfiguration* by showing exactly what happens when two specific attack strings are typed into a search box [1]. Nothing connects to a real database — all SQL, server responses, and query results are simulated so everything is safe to explore [1].

**The core concept:** One search box, two servers [1]. Every time you submit input, it goes to a misconfigured server (red panel) and a properly configured server (green panel) at the same time [1]. You see what each one does differently [1].
![snap1](/EF_Week7_Vipe_Coding_Assign3/Images_W7_Vipe_Coding_3/Snap1_Program.png)

Pic.1 (Educational Program)

---
### Navigation / The Program

The header has two tabs [1]:

| Tab | Attack String |
| :--- | :--- |
| **01 — OR Tautology String** | `OR 1=1` |
| **02 — UNION Extraction String** | `UNION SELECT username,password,email FROM users--` |

The **A02 badge** in the top-left identifies the OWASP category [1]. The **green dot** confirms this is a sandbox — no real server or database is involved [1].

---

### Attack 01 — OR Tautology String

*   **Attack string:** ` OR 1=1`

#### What it does
The opening `'` closes the developer's string literal [1]. Everything after it is now raw SQL [1]. `OR 1=1` is a condition that is always true — so the `WHERE` clause passes every single row in the table [1]. The attacker receives the entire inventory in one request, regardless of what they searched for [1].

#### The misconfiguration
The server concatenates user input directly into SQL: `WHERE product = '` + `input` + `'` [1]. There is no parameterization layer [1]. The database parser cannot tell the difference between the developer's SQL and the attacker's SQL [1].

#### How to use it

| Button | What happens |
| :--- | :--- |
| **Load attack string** | Fills the search box with ` OR 1=1` |
| **Submit** | Both panels respond simultaneously |
| **Normal search: Laptop** | Fills a clean term — both panels return 0 rows identically |
| **Clear** | Empties the box |

*   **Vulnerable panel (red):**  
    The SQL the database receives becomes `WHERE product = '' OR 1=1` [1]. Since `1=1` is always true, all 4 inventory rows are returned — the entire table dumped in one search [1].
*   **Secure panel (green):**  
    The query template is compiled first — `WHERE product = $1` — and the payload is passed as a data parameter [1]. The database searches for a product literally named `' OR 1=1` [1]. It finds none. 0 rows returned [1]. The apostrophes and `OR` are treated as string characters, not SQL operators [1].

#### Expandable sections
*   **How attackers use this in practice:** Shows how OR injection is quickly escalated to a UNION attack that extracts data from other tables like users or sessions [1].
*   **Fix:** Parameterized query code in Python, Node.js, and Java [1].

![sanp2](/EF_Week7_Vipe_Coding_Assign3/Images_W7_Vipe_Coding_3/Snap2_String1.png)

Pic.2 (String Attack 01)


![snap4](/EF_Week7_Vipe_Coding_Assign3/Images_W7_Vipe_Coding_3/Snap4_Fix1.png)

Pic.3 (Fix String Attack 01)

---

### Attack 02 — UNION Extraction String

*   **Attack string:** ` UNION SELECT username,password,email FROM users--`

#### What it does
The `'` closes the original query string [1]. `UNION SELECT` appends a completely separate query targeting the `users` table [1]. `--` comments out the trailing quote so the syntax stays valid [1]. If the database user has permission to read the `users` table, the credentials are merged into the response and returned alongside the (empty) product results — a complete account list in a single search request [1].

#### The misconfiguration — two failures, not one
1.  **String concatenation:** Input reaches the SQL parser as code, not data [1].
2.  **Overprivileged DB user:** The app's database user has `SELECT ON ALL TABLES` instead of only the tables this feature needs [1]. Even if injection happened, a least-privilege DB user would block it at the database engine level [1].

#### How to use it
Same buttons as Attack 01 [1]. The key difference is what appears on the vulnerable side when the attack string is submitted [1].

*   **Vulnerable panel (red):**  
    The database merges two result sets — the product columns aligned with `username`, `password` (bcrypt hash), and `email` columns from the `users` table [1]. Three admin credential rows are returned in the HTTP response [1].
*   **Secure panel (green):**  
    Two independent defenses:
    1.  The parameterized query makes `UNION` inert — it is a literal string passed as `$1`, not SQL syntax [1].
    2.  Even if parameterization had a gap, the DB role `app_searchonly` has `SELECT ON inventory` [1]. The database engine itself blocks the query with permission denied for table `users` [1].

#### Expandable sections
*   **How attackers use this in practice:** Shows how attackers first probe the column count with `ORDER BY`, then construct a precisely matching `UNION` to extract any table [1].
*   **Fix:** PostgreSQL `CREATE ROLE` and `GRANT` commands for setting up least-privilege roles, one per app feature [1].

![snap3](/EF_Week7_Vipe_Coding_Assign3/Images_W7_Vipe_Coding_3/SnaP3_String2.png)

Pic.4 (String Attack 02)

![snap5](/EF_Week7_Vipe_Coding_Assign3/Images_W7_Vipe_Coding_3/Snap5_Fix2.png)

Pic5 (Fix String Attack 02)

---

### Comparison Table

Both attack strings are shown side-by-side with four columns — the attack string itself, its mechanism, its impact, and the fix — so learners can compare them at a glance without switching tabs [1].

| | Attack 01 | Attack 02 |
| :--- | :--- | :--- |
| **String** | `OR 1=1` | `UNION SELECT …` |
| **Mechanism** | Makes `WHERE` always true | Appends a second `SELECT` to a different table |
| **Impact** | Dumps entire table immediately | Extracts credentials silently |
| **Fix** | Parameterized query | Parameterized query + least-privilege DB role |
