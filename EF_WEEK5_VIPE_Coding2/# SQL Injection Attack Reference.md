# SQL Injection Attack Reference
### OWASP A05:2025 — Injection | Security Vulnerability Lab (Week 5 Vipe Coding 2)

*Course*: MSSE 642 – Software Assurance  
*Student*: EMAD FATTAH  
*Date:* June 11, 2026
---

## Introduction

### What Is SQL Injection?

SQL Injection (SQLi) is one of the oldest, most well-known, and most dangerous vulnerabilities in web application security. It has appeared in the **OWASP Top 10** list of critical security risks for over two decades and remains ranked **#5 in the 2025 edition** — not because defenders don't know about it, but because it continues to be found in real systems every day.

At its core, SQL injection happens when an application builds a database query by **directly concatenating user input into a SQL string** without separating the data from the SQL code. This blurs the boundary between *instructions* and *data* — allowing an attacker to submit input that is interpreted as SQL commands rather than as a search term or value.

```sql
-- What the developer intended:
SELECT * FROM products WHERE name ILIKE '%laptop%'

-- What the attacker sends instead:
SELECT * FROM products WHERE name ILIKE '%' OR '1'='1%'
```

The database cannot tell the difference. It executes whatever it receives.

---

### Why It Still Happens

Despite being widely understood, SQL injection persists for several reasons:

- **Legacy codebases** — older applications built before parameterized queries became standard practice.
- **Developer oversight** — a single unparameterized query in a large codebase is enough to create a critical vulnerability.
- **Rapid development pressure** — shortcuts taken under deadline pressure that never get fixed.
- **Third-party components** — plugins, libraries, or CMS extensions that introduce vulnerabilities outside the developer's direct control.
- **No visible error** — a vulnerable query often works perfectly for normal users, so it goes undetected until an attacker finds it.

---

### What an Attacker Can Do

Depending on the database configuration and query structure, a successful SQL injection attack can allow an attacker to:

| Capability | Example |
|---|---|
| **Bypass authentication** | Log in as any user without a password |
| **Read sensitive data** | Extract usernames, emails, and password hashes |
| **Read hidden columns** | Access fields the application never displays |
| **Query other tables** | Steal data from tables unrelated to the original query |
| **Infer data blindly** | Extract secrets character-by-character using timing |
| **Destroy data** | Drop tables, delete records, wipe entire databases |
| **Escalate privileges** | Identify admin accounts and target them specifically |

All of these are possible through a single vulnerable input field — a search box, a login form, a URL parameter, even an HTTP header.

---

### The Scale of Real-World Impact

SQL injection has been responsible for some of the largest data breaches in history:

- **Heartland Payment Systems (2008)** — 130 million credit card numbers stolen via SQL injection. The attacker, Albert Gonzalez, was sentenced to 20 years in federal prison. ¹
- **Sony Pictures (2011)** — over 1 million user accounts compromised, including plaintext passwords, email addresses, and dates of birth, extracted through a single SQL injection flaw. ²
- **Yahoo! (2012)** — 450,000 credentials exposed through a SQLi vulnerability in Yahoo! Voices. Passwords were stored unsalted in plaintext. ³
- **TalkTalk (2015)** — 157,000 customer records and 15,600 bank account numbers stolen. The attacker was 17 years old, using publicly documented SQL injection tools. The breach cost TalkTalk £77 million. ⁴

These were not sophisticated nation-state attacks. They were opportunistic exploits of a well-known, preventable flaw.

---

#### References

¹ Zetter, K. (2010, March 26). *TJX Hacker Gets 20 Years in Prison.* Wired.
https://www.wired.com/2010/03/gonzalez-sentencing

² Poulsen, K. (2011, June 2). *Sony Pictures Hacked, Exposing 1 Million Accounts.* Wired.
https://www.wired.com/2011/06/sony-pictures-hacked

³ Gallagher, S. (2012, July 12). *Yahoo data breach: 450,000 credentials leaked.* Ars Technica.
https://arstechnica.com/information-technology/2012/07/yahoo-data-breach-450000-credentials-leaked

⁴ Information Commissioner's Office. (2016, October 5). *TalkTalk Telecom Group PLC — Monetary Penalty Notice.* ICO.
https://ico.org.uk/action-weve-taken/enforcement/talktalk-telecom-group-plc

---

**Further Reading**

- OWASP — SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
- OWASP Top 10:2025 — A05 Injection: https://owasp.org/Top10/A03_2021-Injection
- CWE-89 — SQL Injection: https://cwe.mitre.org/data/definitions/89.html
- NIST NVD — SQL Injection vulnerability database: https://nvd.nist.gov/vuln/search/results?query=sql+injection
- PortSwigger Web Security Academy — SQL Injection: https://portswigger.net/web-security/sql-injection

---

### The Defence — Parameterized Queries

The primary defence against SQL injection is **parameterized queries** (also called prepared statements). Instead of building a SQL string by concatenation, the query structure is defined first and the user's data is passed separately:

```sql
-- Vulnerable: data mixed with SQL structure
"SELECT * FROM products WHERE name ILIKE '%" + userInput + "%'"

-- Secure: structure and data are always separate
"SELECT * FROM products WHERE name ILIKE $1"
-- with parameter: ["%" + userInput + "%"]
```

The database receives the SQL structure and the data through different channels. No matter what the user types — quotes, semicolons, SQL keywords — it is always treated as a literal value. It can never be interpreted as code.

Supporting defences include:

- **Input validation** — reject or flag inputs containing SQL metacharacters (`'`, `;`, `--`)
- **Least-privilege database accounts** — the app's DB user should only have the permissions it actually needs (e.g. `SELECT` only — never `DROP`)
- **Web Application Firewalls (WAF)** — detect and block common injection patterns at the network level
- **Security logging (A09:2025)** — record every query attempt so attacks are detected even when they fail
- **Error handling** — never expose raw database error messages to users; they leak system internals

---

### How This Lab Is Structured

This lab demonstrates SQL injection hands-on through **six attack types**, each targeting a different technique an attacker might use. Every attack is run against two endpoints side by side:

- **Vulnerable Endpoint** — builds queries with string concatenation. No logging. Attacks succeed silently.
- **Secure Endpoint** — uses parameterized queries. Every attempt is logged and classified by severity.

The contrast between the two is the lesson: the same input, two different outcomes, one line of code difference.

Work through each attack in order. Try the payload in the Vulnerable search box first — observe what comes back. Then try the same payload in the Secure search box and watch it get blocked and logged.

![SQL Injection Lab — Landing Page]()
---

## Attack 1 — Boolean-Based Bypass

**Payload:** `' OR '1'='1`

### What the normal query looks like
When you type `laptop` into the search box, the vulnerable endpoint builds this:
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%laptop%'
   OR category ILIKE '%laptop%'
```

### What the attack does
When you type `' OR '1'='1`, that input is dropped directly into the string:
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%' OR '1'='1%'
   OR category ILIKE '%' OR '1'='1%'
```

- **`'`** — closes the string literal. You've escaped from data into SQL code.
- **`OR '1'='1`** — adds a condition that is *always true*. Since `'1'='1'` is never false, every single row in the table matches.
- The `WHERE` clause now returns everything — including the `secret` column and any data never meant to be visible.

### Why the secure endpoint blocks it
```
SQL:   WHERE name ILIKE $1 OR category ILIKE $1
Data:  $1 = "%' OR '1'='1%"
```
PostgreSQL receives the single quote as *data to search for*, not SQL syntax. It literally searches for a product whose name contains the string `' OR '1'='1` — finds nothing — and returns zero rows.

### Key lesson
The vulnerability isn't the search box — it's **trusting user input as code**. Parameterized queries enforce a hard boundary: SQL structure goes one way, data goes another, and they can never mix.

---

## Attack 2 — UNION Data Extraction

**Payload:** `x' UNION SELECT id, username, email, password_hash, role FROM lab_users--`

### What UNION does
`UNION` joins the results of two `SELECT` statements into one result set. Both queries must return the same number of columns with compatible types. Attackers exploit this to bolt a second query onto the original one — querying a completely different table.

### What the attack does
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%x' UNION SELECT id, username, email, password_hash, role FROM lab_users--%'
```

- **`x'`** — `x` is a dead search term (no product is named "x"). The `'` closes the string literal.
- **`UNION SELECT ... FROM lab_users`** — appends a full second query against the users table. The column count (5) matches the original query's 5 columns.
- **`--`** — a SQL comment that discards the original closing `%'`, preventing a syntax error.

**Result:** zero product rows + every row from `lab_users` — usernames, emails, and bcrypt password hashes — returned through a normal search box.

### Why this is dangerous

| What the attacker learns | How they use it |
|---|---|
| Usernames | Target accounts for phishing or credential stuffing |
| Email addresses | Sell them, spam them, or use them to reset passwords |
| Password hashes | Run offline cracking tools to recover real passwords |
| Roles | Know which accounts are admins to target next |

### Key lesson
A normal search box can become a full database dump tool if queries are built by string concatenation. Parameterized queries mean the UNION is never executed — `$1` receives the entire payload as a literal string.

---

## Attack 3 — Comment-Based Tautology

**Payload:** `x' OR 1=1--`

### What a tautology is
A **tautology** is a statement that is always true by definition. In SQL, `1=1` is a tautology — it can never be false, regardless of the data in the table.

### What the attack does
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%x' OR 1=1--%'
   OR category ILIKE '%x' OR 1=1--%'
```

- **`x'`** — `x` is a dead search term. The `'` escapes into SQL code.
- **`OR 1=1`** — permanently true condition. Every row in the table satisfies the `WHERE` clause.
- **`--`** — SQL line comment. Discards the trailing `%'` from the original query, preventing a syntax error and concealing the injection.

### How this differs from Boolean Bypass

| | Boolean Bypass | Comment Tautology |
|---|---|---|
| Payload | `' OR '1'='1` | `x' OR 1=1--` |
| Trick | Compares strings | Compares integers |
| Comment needed? | No | **Yes** — silences the broken query |
| Detectability | Slightly more obvious | `--` hides errors from logs |

The `--` comment is what makes this technique effective in the real world — the injected query is syntactically clean, no error is raised, and nothing looks wrong from the outside.

### Key lesson
`--` is the attacker's cleanup tool. It lets them inject new SQL logic while silently discarding whatever was left of the original query, keeping the final statement syntactically valid.

---

## Attack 4 — Time-Based Blind Injection

**Payload:** `'; SELECT pg_sleep(3)--`

### What "blind" means
Most SQL injection attacks rely on seeing results — rows come back, or an error message leaks data. **Blind injection** is different: the page shows nothing useful. No rows, no errors, no data. The attacker is working completely in the dark.

Time-based blind injection solves this by using the database as a **stopwatch**.

### What the attack does
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%'; SELECT pg_sleep(3)--%'
```

- **`'`** — closes the string literal.
- **`;`** — ends the first statement. This starts a brand new SQL statement (**stacked query**).
- **`SELECT pg_sleep(3)`** — tells PostgreSQL to pause for exactly 3 seconds before responding.
- **`--`** — comments out the leftover `%'`.

### Why a delay is useful
On its own, `pg_sleep(3)` doesn't leak data. The power comes from **conditional logic**:
```sql
'; SELECT CASE WHEN (SUBSTRING(password,1,1)='a') THEN pg_sleep(3) ELSE pg_sleep(0) END
FROM lab_users WHERE username='admin'--
```

- Response takes **3 seconds** → condition is true (first character is `a`)
- Response is **instant** → condition is false (try the next character)

Repeat for every character, every column, every table. Automated tools like **sqlmap** can reconstruct an entire database using only response time as a signal.

### Key lesson
A site that shows **no output at all** is not safe from SQL injection. As long as the query is vulnerable, response *time* leaks information — and that's all an automated tool needs to reconstruct every secret in your database, one millisecond at a time.

---

## Attack 5 — Error-Based Extraction

**Payload:** `' AND 1=CAST(version() AS INT)--`

### The core idea
Instead of reading data through query results, error-based injection **deliberately crashes the database** in a way that forces the error message itself to contain the data you want. The error message becomes the data channel.

### What the attack does
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%' AND 1=CAST(version() AS INT)--%'
```

- **`'`** — closes the string literal.
- **`CAST(version() AS INT)`** — `version()` returns a text string like `PostgreSQL 16.2 on x86_64-pc-linux-gnu`. Casting that to an integer **always fails**.
- PostgreSQL raises a type conversion error and includes the original string value in the message:
  ```
  ERROR: invalid input syntax for type integer: "PostgreSQL 16.2 on x86_64-pc-linux-gnu..."
  ```
- **`--`** — comments out the rest.

### What else can be extracted
`version()` is just the simplest example. Attackers swap it out for anything:
```sql
-- Extract the current database user
' AND 1=CAST(current_user AS INT)--

-- Extract a password hash
' AND 1=CAST((SELECT password_hash FROM lab_users LIMIT 1) AS INT)--

-- Extract any column from any table
' AND 1=CAST((SELECT email FROM lab_users WHERE role='admin') AS INT)--
```

### The two conditions that make this work
1. The app must display **raw database errors** to the user.
2. The query must be **vulnerable to injection**.

Remove either condition and the attack fails.

### Key lesson
**Never show raw database error messages to users.** Even a "harmless" error can hand an attacker your database version, OS details, and the contents of your tables. Production apps must use generic error pages, with real errors written only to internal logs.

---

## Attack 6 — Stacked / Destructive Query (DDL)

**Payload:** `'; DROP TABLE products--`

### What DDL means
**DDL** (Data Definition Language) is the category of SQL commands that change the *structure* of a database — `DROP TABLE`, `CREATE TABLE`, `ALTER TABLE`, `TRUNCATE`. They don't query rows, they destroy or reshape the tables those rows live in.

### What the attack does
```sql
SELECT id, name, category, price, secret
FROM products
WHERE name ILIKE '%'; DROP TABLE products--%'
```

- **`'`** — closes the string literal.
- **`;`** — ends the current SQL statement and starts a new, completely independent one (**stacked query**).
- **`DROP TABLE products`** — permanently and irreversibly deletes the entire table — every row, every column, the table definition itself — in a single operation.
- **`--`** — comments out the trailing `%'`.

In a real vulnerable system both statements execute in sequence. The search returns zero results, and the table is gone.

### How this differs from every other attack

| Attack | Goal | Reversible? |
|---|---|---|
| Boolean Bypass | Read all rows | Yes — no data changed |
| UNION Extraction | Steal rows from another table | Yes — no data changed |
| Comment Tautology | Read all rows | Yes — no data changed |
| Time-Based Blind | Infer data via timing | Yes — no data changed |
| Error-Based | Leak data via error messages | Yes — no data changed |
| **Stacked DDL** | **Destroy the table** | **No — permanent** |

This is the **"Little Bobby Tables"** attack — named after the famous xkcd comic where a mother names her son `Robert'); DROP TABLE Students;--` and wipes a school's database.

> _"And the teachers never heard from little Bobby Tables again."_

### Defense-in-depth
Even with parameterized queries, the right answer is to layer multiple defenses:

1. **Parameterized queries** — prevent the injection in the first place
2. **Least-privilege DB accounts** — the app's DB user should only have `SELECT` on the tables it needs, never `DROP` or DDL privileges
3. **Automated backups** — if a destructive query somehow executes, you can restore. Without backups, the data is gone forever.

### Key lesson
SQL injection without backups and without least-privilege accounts isn't just a data theft risk — it's an **existential risk** to the application. A single unsanitized input can permanently destroy everything.

---

## Summary

| # | Attack | Payload Pattern | What It Does |
|---|---|---|---|
| 1 | Boolean Bypass | `' OR '1'='1` | Returns all rows by making WHERE always true |
| 2 | UNION Extraction | `x' UNION SELECT ... FROM lab_users--` | Steals data from a completely different table |
| 3 | Comment Tautology | `x' OR 1=1--` | Returns all rows; `--` hides the broken query |
| 4 | Time-Based Blind | `'; SELECT pg_sleep(3)--` | Leaks data through response timing — no output needed |
| 5 | Error-Based | `' AND 1=CAST(version() AS INT)--` | Forces a crash that leaks data in the error message |
| 6 | Stacked DDL | `'; DROP TABLE products--` | Destroys the table permanently |

**The single defense that stops all six:** parameterized queries (prepared statements).
