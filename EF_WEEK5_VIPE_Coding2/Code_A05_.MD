## 
Here's how the SQL side of the lab is structured:
## The Two Tables

* products — what students search against. Columns: id, name, category, price, and secret (a hidden field simulating sensitive data that shouldn't be reachable through a normal search).
* lab_users — a separate table holding id, username, email, password_hash, and role (p. 1). Students never interact with it directly — it exists to be stolen via UNION injection.

## The Core Contrast — How Each Endpoint Builds Its Query## Vulnerable endpoint (string concatenation)

SELECT id, name, category, price::text, secretFROM productsWHERE name ILIKE '%{user_input}%'
 OR category ILIKE '%{user_input}%'

The user's text is dropped straight into the string. A single quote ' in the input escapes the SQL string literal, letting the attacker rewrite the query logic entirely.
## Secure endpoint (parameterized query)

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE $1 OR category ILIKE $1-- param: "%{user_input}%"

The $1 is a placeholder (p. 1). PostgreSQL receives the SQL structure and the data separately — the input can never be interpreted as SQL code, only as a literal value.
## The 6 Attacks and What Each One Exploits

| # | Attack | Technique |
|---|---|---|
| 1 | Boolean Bypass | ' OR '1'='1 — makes the WHERE clause always true, dumps all rows|
| 2 | UNION Extraction | Appends a SELECT against lab_users to steal usernames + password hashes|
| 3 | Comment Tautology | OR 1=1-- — -- discards the rest of the original query, silencing syntax errors|
| 4 | Time-Based Blind | pg_sleep(3) — even with no visible output, a 3-second delay confirms injection works|
| 5 | Error-Based Extraction | Forces a type cast failure; PostgreSQL's error message leaks the database version string|
| 6 | Stacked DDL | ; DROP TABLE products-- — blocked by the server before execution to protect lab data|

------------------------------
## The Logging Layer (A09:2025)

* The vulnerable endpoint logs nothing — attacks succeed silently, leaving no trace.
* The secure endpoint logs every search: timestamp, IP address, the raw query, whether injection was detected, and a severity level (info / warning / critical). Those entries appear live in the Security Event Log panel.

This directly teaches OWASP A09:2025 (Security Logging Failures) — the difference between a system that gets attacked invisibly versus one that detects, classifies, and records every attempt in real time.
## 1. Boolean-Based Bypass — ' OR '1'='1## What the normal query looks like
When you type laptop into the search box, the vulnerable endpoint builds this:

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%laptop%'
 OR category ILIKE '%laptop%'

Only rows where the name or category contains "laptop" are returned.
## What the attack does
When you type ' OR '1'='1, that input is dropped directly into the string:

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%' OR '1'='1%'
 OR category ILIKE '%' OR '1'='1%'

Breaking it down character by character:

* ' — the single quote closes the string literal the SQL was already in. You've escaped out of the data zone into the SQL code zone.
* OR '1'='1 — adds a condition that is always true. '1'='1' is never false.
* The WHERE clause now reads: "return this row IF the name matches… OR if 1=1" — since 1=1 is always true, every single row in the table matches.

The result: the database returns everything — including the secret column, hidden records, and any data that was never meant to be visible through the search box.
------------------------------
## Why the secure endpoint blocks it
With a parameterized query, the $1 placeholder is sent to PostgreSQL separately from the SQL structure:

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%' OR '1'='1%"

PostgreSQL receives the single quote as data to search for, not as SQL syntax. It literally searches for a product whose name contains the string ' OR '1'='1 — finds nothing — and returns zero rows. The quote never gets a chance to break out.
## The key lesson
The vulnerability isn't the search box — it's trusting user input as code (p. 3). String concatenation blurs the line between SQL structure and SQL data (p. 3). Parameterized queries enforce a hard boundary: structure goes one way, data goes another, and they can never mix.
## 2. UNION Data Extraction — x' UNION SELECT id, username, email, password_hash, role FROM lab_users--## What UNION does in SQL
UNION joins the results of two SELECT statements into one result set. The rule is: both queries must return the same number of columns with compatible types. Attackers exploit this to bolt a second, completely different query onto the original one.
## How the attack is built piece by piece
The payload is: x' UNION SELECT id, username, email, password_hash, role FROM lab_users--
The vulnerable endpoint produces this SQL:

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%x' UNION SELECT id, username, email, password_hash, role FROM lab_users--%'

------------------------------
## 
Breaking it down:

* x' — the x is a deliberate dead search term (no product is named "x"). The ' closes the string literal, escaping into SQL code.
* UNION SELECT id, username, email, password_hash, role FROM lab_users — appends a full second query against the lab_users table. The attacker matches the column count (5) to the original query's 5 columns so the types line up.
* -- — a SQL line comment. Everything after it (the original closing %') is discarded, preventing a syntax error.

The combined result set now contains (p. 4):

* Rows from products where name = "x" → zero rows (nothing named x)
* Rows from lab_users → all users, all columns

What comes back in the search results: every username, email address, and bcrypt password hash in the database.
## Why this is one of the most dangerous attacks

| What the attacker learns | How they use it |
|---|---|
| Usernames | Target accounts for phishing or credential stuffing |
| Email addresses | Sell them, spam them, or use them to reset passwords |
| Password hashes | Run offline cracking tools (hashcat, John the Ripper) to recover real passwords  |
| Roles | Know which accounts are admins to target next  |

And all of this comes back through a normal search box — no login required, no special access .
## Why the secure endpoint blocks it completely
With a parameterized query :

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%x' UNION SELECT id, username, email, password_hash, role FROM lab_users--%"

PostgreSQL searches for a product whose name literally contains the string x' UNION SELECT… — finds nothing — and returns zero rows . The UNION never executes because the single quote is data, not syntax . The lab_users table is never touched .
## The key lesson
A normal search box can become a full database dump tool if the query is built by string concatenation (p. 4). UNION attacks require no prior knowledge of the system — an attacker just needs to figure out the column count (easily done by trial and error) and then they can query any table they want.
------------------------------
 3. Comment-Based Tautology — x' OR 1=1--## What is Tautology
A tautology is a statement that is always true by definition . In SQL, 1=1 is a tautology — it can never be false, no matter what data is in the table .
## How the attack is constructed
The payload is: x' OR 1=1-- 
The vulnerable endpoint produces this SQL :

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%x' OR 1=1--%'
   OR category ILIKE '%x' OR 1=1--%'

Three moving parts :

* x' — x is a dead search term . The ' closes the string literal, escaping from data into SQL code .
* OR 1=1 — injects a condition that is permanently true . The WHERE clause now reads: "return this row if the name matches x… OR if 1 equals 1." Since 1 always equals 1, every row in the table satisfies the condition .
* -- — a SQL line comment . Everything after it on that line is ignored by the database . This discards the trailing %' from the original query, which would have caused a syntax error and given the attack away .

Result: all rows returned, including the secret column.
## How this differs from Boolean Bypass (Attack 1)
Both attacks return all rows — but they do it differently:

| Feature | Boolean Bypass | Comment Tautology |
|---|---|---|
| Payload | ' OR '1'='1 | x' OR 1=1-- (p. 5) |
| Trick | Breaks the string with ' and compares strings | Breaks the string and compares integers |
| Comment needed? | No — closes the string naturally | Yes — -- is required to silence the leftover %'  |
| Detectability | Slightly more obvious | The -- hides the broken query from error logs  |

The -- comment is what makes this technique particularly useful in the real world — it makes the injected query syntactically clean, meaning no database error is raised, and the attack is harder to spot by inspecting error logs .
------------------------------
 Why the secure endpoint blocks it
With a parameterized query :

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%x' OR 1=1--%"

PostgreSQL searches for a product literally named x' OR 1=1-- . The single quote, the OR, and the -- are all just characters inside a search string . The tautology never executes . Zero rows returned .
## The key lesson
The -- comment is the attacker's cleanup tool . It lets them inject new SQL logic while silently discarding whatever was left of the original query — keeping the final statement syntactically valid so no error is raised and nothing looks wrong from the outside .
## 4. Time-Based Blind Injection — '; SELECT pg_sleep(3)--## What "blind" means
Most SQL injection attacks rely on seeing the results — a table of rows comes back, or an error message leaks data . Blind injection is different: the page shows nothing useful . No rows, no errors, no data . The attacker is working completely in the dark .
Time-based blind injection solves this by using the database as a stopwatch .
## How the attack works
The payload is: '; SELECT pg_sleep(3)-- 
The vulnerable endpoint produces this SQL :

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%'; SELECT pg_sleep(3)--%'

Breaking it down :

* ' — closes the string literal, escaping into SQL code .
* ; — ends the first statement entirely (p. 6). This is a stacked query — it starts a brand new SQL statement .
* SELECT pg_sleep(3) — pg_sleep(3) tells PostgreSQL to pause for exactly 3 seconds before responding .
* -- — comments out the leftover %' to keep syntax clean .

------------------------------
## 
The server sits frozen for 3 seconds, then responds . In the lab, this delay is simulated server-side to demonstrate the effect safely .
## Why a delay is useful
On its own, pg_sleep(3) doesn't leak data . The power comes from combining it with conditional logic :

-- "Is the first character of the admin password 'a'?"'; SELECT CASE WHEN (SUBSTRING(password,1,1)='a') THEN pg_sleep(3) ELSE pg_sleep(0)END FROM lab_users WHERE username='admin'--


* If the response takes 3 seconds → the condition is true (first character is 'a') 
* If the response is instant → the condition is false (try the next character) 

Repeat this for every character, every column, every table . Automated tools like sqlmap can reconstruct an entire database this way using only response time as a signal — no visible output needed .
## What "blind" looks like in practice

| What the attacker observes | What it means |
|---|---|
| Response takes ~3 seconds | Injection is working; condition was true  |
| Response is instant | Condition was false  |
| Response always takes 3 seconds | Injection confirmed but condition always true — refine the query  |

A skilled attacker can extract a full password hash character by character in minutes using this technique, even against a site that displays nothing but a blank page .
## Why the secure endpoint blocks it

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%'; SELECT pg_sleep(3)--%"

The semicolon, pg_sleep, and everything else are just characters inside a search string . PostgreSQL never executes a second statement . The response comes back instantly .
## The key lesson
A site that shows no output at all is not safe from SQL injection . As long as the query is vulnerable, the database's response time leaks information — and that's all an automated tool needs to reconstruct every secret in your database, one millisecond at a time .
## 5. Error-Based Extraction — ' AND 1=CAST(version() AS INT)--## The core idea
------------------------------
## 
Most attacks try to see data through query results . Error-based injection takes a different route: it deliberately crashes the database in a way that forces the error message itself to contain the data you want to steal (p. 8). The error message becomes the data channel .
## How the attack is constructed
The payload is: ' AND 1=CAST(version() AS INT)-- 
The vulnerable endpoint produces this SQL :

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%' AND 1=CAST(version() AS INT)--%'

Breaking it down :

* ' — closes the string literal, escaping into SQL code .
* AND 1=CAST(version() AS INT) — here's the trap :
* version() is a built-in PostgreSQL function that returns a text string like PostgreSQL 16.2 on x86_64-pc-linux-gnu 
   * CAST(... AS INT) tries to convert that text into an integer 
   * That conversion always fails — you can't turn "PostgreSQL 16.2..." into a number 
   * PostgreSQL raises a type conversion error and includes the original string value in the message 
* -- — comments out the rest to keep syntax clean .

The error message that comes back looks like :

ERROR: invalid input syntax for type integer: "PostgreSQL 16.2 on x86_64-pc-linux-gnu, compiled by gcc..."

The database version, OS, compiler details — all leaked in a single error .
## Why this is more powerful than it looks
version() is just the simplest example . Attackers swap it out for anything :

-- Extract the current database user
' AND 1=CAST(current_user AS INT)--
-- Extract a password hash from another table
' AND 1=CAST((SELECT password_hash FROM lab_users LIMIT 1) AS INT)--
-- Extract any column from any table' AND 1=CAST((SELECT email FROM lab_users WHERE role='admin') AS INT)--

Each one forces a type error that carries the real data value back in the error message . No UNION needed, no timing analysis — just a crash with a useful message .
## The two conditions that make this work
------------------------------
## 

   1. The app must display raw database errors to the user (p. 9). Sites in development mode often do this (p. 9). Production sites sometimes forget to disable it .
   2. The query must be vulnerable to injection (string concatenation, no parameterization) (p. 9).

If either condition is removed, the attack fails (p. 9).
## Why the secure endpoint blocks it

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%' AND 1=CAST(version() AS INT)--%"

The single quote, CAST, version() — all treated as a literal search string . PostgreSQL never evaluates version() as a function call . No error is raised (p. 9). No data leaks .
## The key lesson
Never show raw database error messages to users . Even a "harmless" error message can hand an attacker your database version, OS details, and the content of your tables . Production apps must use generic error pages — "something went wrong" — with the real error written only to internal server logs .
## 6. Stacked DDL — '; DROP TABLE products--## What DDL means
DDL stands for Data Definition Language — the category of SQL commands that change the structure of a database rather than its data . Commands like DROP TABLE, CREATE TABLE, ALTER TABLE, and TRUNCATE are all DDL . They don't query rows — they destroy or reshape the tables those rows live in .
## How the attack is constructed
The payload is: '; DROP TABLE products-- 
The vulnerable endpoint produces this SQL :

SELECT id, name, category, price, secretFROM productsWHERE name ILIKE '%'; DROP TABLE products--%'

Breaking it down :

* ' — closes the string literal, escaping into SQL code .
* ; — this is the key ingredient . A semicolon ends the current SQL statement and starts a new, completely independent one . This is called a stacked query .
* DROP TABLE products — the second statement . It permanently and irreversibly deletes the entire products table — every row, every column, the table definition itself — in a single operation.

------------------------------
##

* -- — comments out the trailing %' to prevent a syntax error .

In a real vulnerable system, the database would execute both statements in sequence . The search returns zero results, and the table is gone .
## What makes this different from the other attacks
Every other attack in the lab reads data . Stacked DDL destroys it .

| Attack | Goal | Reversible? |
|---|---|---|
| Boolean Bypass | Read all rows | Yes — no data changed  |
| UNION Extraction | Steal rows from another table | Yes — no data changed  |
| Comment Tautology | Read all rows | Yes — no data changed  |
| Time-Based Blind | Infer data via timing | Yes — no data changed  |
| Error-Based | Leak data via error messages | Yes — no data changed  |
| Stacked DDL | Destroy the table | No — permanent  |

This is the "Little Bobby Tables" attack — named after the famous xkcd comic where a mother names her son Robert'); DROP TABLE Students;-- and wipes a school's database .
## Why the lab blocks it
The server detects the DROP TABLE pattern before the query runs and returns a warning instead of executing it :

CATASTROPHIC INJECTION DETECTED: This payload would have executed a destructive 
SQL statement against your database in a real system. Blocked in this lab to 
preserve your data.

This is intentional — if the lab actually executed it, you'd lose all the product data and the other attacks would stop working . In a real vulnerable app, there is no such safety net .
## Why the secure endpoint blocks it

SQL:  WHERE name ILIKE $1 OR category ILIKE $1Data: $1 = "%'; DROP TABLE products--%"

The semicolon is just a character inside a search string . PostgreSQL never sees a second statement . No second statement means no DROP . The table is never touched .
## The key lesson — and the defense-in-depth mindset
Even with parameterized queries (which prevent injection), the right answer is to layer multiple defenses :

   1. Parameterized queries — prevent the injection in the first place 

------------------------------
## 

   1. Least-privilege database accounts — the app's DB user should only have SELECT on the tables it needs, never DROP or DDL privileges 
   2. Automated backups — if somehow a destructive query executes, you can restore . Without backups, the data is gone forever .

SQL injection without backups and without least-privilege accounts isn't just a data theft risk — it's an existential risk to the application .
------------------------------
I have successfully parsed the document into Markdown formatting. If you need any assistance explaining specific SQL injection logic, writing remediations, or generating defense strategies, let me know how you would like to proceed!

