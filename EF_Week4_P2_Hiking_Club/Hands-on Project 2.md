# HANDS-ON PROJECT #2: Vulnerability Analysis and Threat Model

**File Path:** `assignments/weekly-projects/project-2-threat-analysis.md`  
**Version:** 3.6  
**Date:** June 3, 2026  
**Author/Group:** [Insert Group Names / GitHub Handles]

---

# Part 1: Secure Design Document Overview

## 1. High-Level Project Description
The Hiking Club Application is a web-based platform designed to streamline event management, member coordination, and financial tracking for outdoor recreational groups. It enables guests to browse upcoming trips, allows registered members to manage their profiles and sign up for events, and provides trip leaders with comprehensive tools to create events, manage attendee lists (including waitlists), and log completion statuses. Additionally, system administrators can manage user accounts, run database integrity checks, and utilize a secure treasury portal to oversee cash flows and process payments for paid excursions. By centralizing these diverse workflows, the application simplifies administrative overhead while fostering community engagement.

## 2. Organization Description
The application will be utilized by **The Summit Trailblazers Hiking Club (STHC)**, a regional non-profit organization with over 1,500 active members, 50 volunteer trip leaders, and a small administrative staff. STHC organizes weekly hikes, multi-day backpacking trips, and educational outdoor workshops. The organization relies on membership dues and individual trip fees to fund its operations, gear rentals, and environmental conservation initiatives. Because the club handles personal contact information, sensitive medical history, and financial transactions, maintaining user trust and data privacy is paramount to their mission.

## 3. Deployment Environment
To ensure high availability, scalability, and robust security, the Hiking Club Application will be deployed in a cloud environment using **Amazon Web Services (AWS)**. 

*   **Front End Web Server:** Hosted on AWS Elastic Container Service (ECS) running Dockerized Node.js/Express containers. These containers reside in a Public Subnet behind an AWS Application Load Balancer (ALB), which handles SSL/TLS termination.
*   **Backend Database Server:** Deployed on AWS Relational Database Service (RDS) running PostgreSQL. The database is isolated within a Private Subnet, completely inaccessible from the public internet.
*   **Network Security:** Security Groups and Network Access Control Lists (NACLs) act as virtual firewalls to restrict traffic. The Database Server will only accept incoming PostgreSQL traffic (Port 5432) from the specific security group assigned to the Front End Web Server.
*   **Outbound Traffic:** A NAT Gateway in the Public Subnet allows the backend systems to download software updates securely without exposing them to inbound internet requests.

## 4. Secure Concepts Applicable to the Hiking Club Application
Based on the project description, several key areas require strict security controls:

*   **Authentication & Session Management:** The system supports three tiers of logged-in users (Members, Trip Leaders, and System Admins). Strong password hashing (using Argon2id), session management (using secure, HTTP-only cookies), and rate-limiting on login endpoints are required to prevent brute-force attacks and session hijacking.
*   **Role-Based Access Control (RBAC):** Proper authorization checks must be enforced at the API level. For example, regular members must not access the treasury portal or medical notes, and Trip Leaders must be restricted from modifying trips created by other leaders.
*   **Data Protection (In Transit and At Rest):** All communication must be encrypted using HTTPS (TLS 1.3). Sensitive database columns, particularly member medical information, private trip leader notes, and payment processing tokens, must be encrypted at rest.
*   **Input Validation & Parameterization:** To prevent injection attacks (SQLi, XSS), all user input from clients must be strictly validated on the server side and database queries must be fully parameterized.
*   **Audit Logging:** Actions taken within the treasury portal (withdrawing funds, processing payments) and administrative changes (creating/disabling users) must generate immutable logs for accountability and compliance.

---

# Part 2: Hiking Club Threat Model Assessment

## Deliverable Part 2A: Architectural Diagram

Below is the architectural diagram showing the systems, networks, trust boundaries, IP addresses, and data flows.

```text
+---------------------------------------------------------------------------------------------------------+
|                                           UNTRUSTED ZONE (Internet)                                     |
|                                                                                                         |
|   +-----------------------+          +-----------------------+          +-----------------------+       |
|   |     Guest Client      |          |     Member Client     |          |     Admin Client      |       |
|   |   (Web Browser/HTML)  |          |   (Web Browser/HTML)  |          |   (Web Browser/HTML)  |       |
|   +-----------+-----------+          +-----------+-----------+          +-----------+-----------+       |
|               |                                  |                                  |                   |
|               | HTTPS (Port 443)                 | HTTPS (Port 443)                 | HTTPS (Port 443)  |
|               v                                  v                                  v                   |
+---------------+----------------------------------+----------------------------------+-------------------+
===========================================================================================================
                                     TRUST BOUNDARY 1: External Firewall (WAF/ALB)
===========================================================================================================
+---------------------------------------------------------------------------------------------------------+
| AWS VPC (10.0.0.0/16)                                                                                   |
|                                                                                                         |
|  +---------------------------------------------------------------------------------------------------+  |
|  | PUBLIC SUBNET (DMZ) - 10.0.1.0/24                                                                 |  |
|  |                                                                                                   |  |
|  |   +-------------------------------------------------------------------------------------------+   |  |
|  |   | FRONT END WEB SERVER                                                                      |   |  |
|  |   | Public IP: 203.0.113.10  |  Private IP: 10.0.1.15                                         |   |  |
|  |   |                                                                                           |   |  |
|  |   |   - Guest Browsing Router      - Member Profile Router      - Admin/Trip Leader Router    |   |  |
|  |   |   - Auth Controller (Bcrypt)   - Event Registration API     - Treasury & Payment Portal   |   |  |
|  |   +---------------------------------------------+---------------------------------------------+   |  |
|  |                                                 |                                                 |  |
|  +-------------------------------------------------|-------------------------------------------------+  |
|                                                    | SQL/TCP (Port 5432)                                |
|                                                    | (Only allowed from 10.0.1.15)                      |
|                                                    v                                                    |
|  =====================================================================================================  |
|                                  TRUST BOUNDARY 2: Internal DB Security Group                           |
|  =====================================================================================================  |
|  +-------------------------------------------------------------------------------------------------+  |
|  | PRIVATE SUBNET - 10.0.2.0/24                                                                      |  |
|  |                                                                                                   |  |
|  |   +-------------------------------------------------------------------------------------------+   |  |
|  |   | BACKEND DATABASE SERVER (PostgreSQL)                                                      |   |  |
|  |   | Private IP: 10.0.2.24 (No Public IP)                                                      |   |  |
|  |   |                                                                                           |   |  |
|  |   |   - Users Table (Hashed Passwords, Profile Data)                                          |   |  |
|  |   |   - Events Table (Trip Details, Max/Min limits)                                           |   |  |
|  |   |   - Registrations Table (Waitlists, Attendee Statuses)                                    |   |  |
|  |   |   - Private Notes & Medical Info Table (Encrypted At Rest)                                |   |  |
|  |   |   - Treasury & Transaction Ledger                                                         |   |  |
|  |   +-------------------------------------------------------------------------------------------+   |  |
|  |                                                                                                   |  |
|  +-------------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------------+
```

### Data Flow Descriptions:
1.  **Guest / Member / Admin to Front End Web Server (HTTPS - Port 443):** Users send HTTPS requests to the Front End Web Server. This includes viewing trips (guests), updating profiles and registering (members), or managing events and treasury data (admins/leaders).
2.  **Front End Web Server to Backend Database Server (SQL/TCP - Port 5432):** The Front End Web Server processes business logic, validates inputs, checks user roles, and executes parameterized SQL queries against the Database Server located securely in the Private Subnet.

---

## Deliverable Part 2B: STRIDE Threat Model

### 1. Spoofing (S)
*   **Threat Description:** An attacker attempts to log in as a System Administrator or a Trip Leader by guessing credentials, launching a brute-force attack, or using credential stuffing. If successful, the attacker can act on behalf of the compromised administrative user.
*   **Impact:** The attacker could disable legitimate user accounts, delete upcoming trips, access confidential medical records, or manipulate the payment portal to redirect treasury funds to their own external accounts.
*   **Mitigation:** Implement strict password complexity requirements, enforce rate-limiting on authentication endpoints (e.g., locking accounts after 5 failed attempts), and mandate Multi-Factor Authentication (MFA) for all Trip Leader and System Admin accounts.

### 2. Tampering (T)
*   **Threat Description:** A malicious member or external attacker intercepts or manipulates HTTP POST requests sent to the server. For example, they might modify the parameter representing the event fee to "$0.00" or change their registration status from "Waitlisted" to "Registered" directly within the request payload.
*   **Impact:** This bypasses business logic, allowing users to register for paid events without paying, or bypass the trip leader's manual selection process from the waitlist, resulting in financial loss and system distrust.
*   **Mitigation:** Never trust client-side data. The server must independently validate the price of the event from the database before processing payments, and strictly enforce that status changes (like moving from Waitlist to Registered) can only be executed by authorized Trip Leaders.

### 3. Repudiation (R)
*   **Threat Description:** A System Admin accesses the treasury portal, withdraws a significant amount of cash to "pay for trip expenses," but later claims they never initiated the transaction. Because the system lacks detailed or secure logging, there is no proof of who performed the action.
*   **Impact:** Financial fraud goes unpunished, and the organization cannot verify internal theft, leading to legal liability and financial ruin for the non-profit.
*   **Mitigation:** Implement an append-only, cryptographically signed audit trail for all financial transactions and administrative changes. Ensure logs capture the user ID, timestamp, IP address, and details of the action, and stream these logs in real-time to a secure, write-once-read-many (WORM) storage service like AWS S3 with Object Lock enabled.

### 4. Information Disclosure (I)
*   **Threat Description:** A regular member crafts a direct API request to fetch private member data (e.g., requesting GET `/api/members/123/notes` instead of their own profile). Alternatively, an attacker intercepts unencrypted HTTP traffic containing sensitive medical information.
*   **Impact:** Exposure of highly confidential medical notes and private assessments written about members by trip leaders. This violates privacy laws and severely damages the club's reputation.
*   **Mitigation:** Enforce strict server-side authorization checks (such as Broken Object Level Authorization / BOLA defenses) ensuring that only authorized Trip Leaders or the specific System Admin can query medical tables. Additionally, enforce TLS 1.3 across all endpoints to prevent packet sniffing.

### 5. Denial of Service (D)
*   **Threat Description:** A malicious actor targets the public-facing Front End Web Server with a Distributed Denial of Service (DDoS) attack, or registers thousands of dummy accounts rapidly to overflow the database connections.
*   **Impact:** The Hiking Club Application becomes completely unresponsive. Legitimate members cannot sign up for high-demand, time-sensitive trips, and trip leaders cannot run reports or manage active events.
*   **Mitigation:** Deploy AWS Shield or Cloudflare in front of the Application Load Balancer to filter out DDoS traffic. Implement CAPTCHA on registration and login pages, and configure database connection pooling with strict timeout limits.

### 6. Elevation of Privilege (E)
*   **Threat Description:** A regular member discovers that by changing a parameter in their profile update request (e.g., sending `"role": "admin"` or `"is_trip_leader": true`), they can elevate their account privileges.
*   **Impact:** The member gains full access to administrative features, including the treasury portal, user management, and the ability to modify or delete any event on the platform.
*   **Mitigation:** The server must explicitly ignore any role-altering parameters sent during standard profile updates. Role assignments must only be modifiable via a dedicated administrative endpoint that is protected by a strict, server-side System Admin authorization check.

---

## Deliverable Part 2C: OWASP Threat Model

Based on the OWASP Application Threat Modeling methodology, the following structured assessment outlines the scope, vulnerabilities, countermeasures, and risk prioritization for the Hiking Club Application.

### 1. Assessment Scope — What’s on the line?
The Hiking Club Application processes several high-value assets that must be protected:
*   **Member Personally Identifiable Information (PII):** Names, emails, phone numbers, emergency contacts, and physical addresses.
*   **Sensitive Medical Records & Private Notes:** Critical health notes (allergies, heart conditions) and private behavioral notes written by trip leaders.
*   **Financial & Treasury Assets:** Access tokens to the payment portal, cash flow logs, and the ability to initiate withdrawals from the treasury.
*   **System Integrity & Availability:** The operational database containing trip schedules, waitlists, and historical event logs.

### 2. Vulnerabilities — What are they?
*   **V1: Broken Object Level Authorization (BOLA / IDOR):** The application relies on user-provided IDs (e.g., `/api/profiles/edit?id=456`) to update profile or medical information without verifying if the logged-in user owns that record.
*   **V2: SQL Injection (SQLi):** Search filters for trips or member directories are constructed using raw string concatenation, allowing attackers to escape the query and read or modify the entire database.
*   **V3: Stored Cross-Site Scripting (XSS):** Trip Leaders can input event descriptions without sanitization. If a malicious leader or compromised account inserts `<script>` tags, the script executes in the browser of any guest or member viewing the trip.
*   **V4: Broken Authentication & Missing MFA:** Administrative accounts do not require Multi-Factor Authentication, and the system does not enforce minimum password length or block common passwords.
*   **V5: Insecure Direct Object References in Treasury:** The treasury portal relies on client-side visibility controls (hiding the button) rather than enforcing server-side API validation for withdrawals.

### 3. Countermeasures — What can you do about it?
*   **C1: Implement Strict Server-Side Authorization Checks (Mitigates V1 & V5):** Every API request must validate the user's session token and confirm that the active role has explicit permission to view or edit the requested resource. Use a decorator pattern or middleware (e.g., `checkRole(['admin', 'trip_leader'])`) on all sensitive routes.
*   **C2: Use Parameterized Queries and ORMs (Mitigates V2):** Utilize a secure Object-Relational Mapper (ORM) like Prisma or Sequelize, and ensure any raw SQL queries use parameterized input placeholders (e.g., `WHERE id = $1`) to prevent query manipulation.
*   **C3: Context-Aware Output Encoding (Mitigates V3):** Sanitize all user-generated content on input using libraries like DOMPurify, and ensure the front-end framework (e.g., React or modern template engines) automatically escapes HTML output before rendering it to the DOM.
*   **C4: Enforce MFA and Strong Password Policies (Mitigates V4):** Integrate AWS Cognito or a dedicated auth library to mandate MFA for Admins and Trip Leaders. Implement password strength checkers (e.g., zxcvbn) and enforce rate-limiting on authentication attempts.
*   **C5: Database Encryption At Rest (Mitigates Data Exposure):** Utilize AWS RDS KMS encryption to encrypt the database storage. For highly sensitive fields (medical records), apply application-level envelope encryption before writing the data to the database.

### 4. Prioritized Risks — Listed in Order of Severity
The following risks are prioritized using a standard Risk Matrix (Likelihood x Impact):

| Rank | Risk Title | Likelihood | Impact | Severity | Primary Countermeasure |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Unauthorized Treasury Withdrawal (Financial Theft)** | Medium | Critical | **Critical** | Implement strict server-side RBAC, transaction limit caps, and mandatory MFA for System Admins. |
| **2** | **SQL Injection leading to Full Database Compromise** | Medium | High | **High** | Enforce parameterized queries and restrict database user privileges to the minimum required. |
| **3** | **Exposure of Sensitive Medical Records (BOLA/IDOR)** | High | High | **High** | Enforce server-side ownership validation checks on all member-specific endpoints. |
| **4** | **Privilege Escalation from Member to Trip Leader/Admin** | Medium | High | **High** | Remove role-assignment capabilities from general profile update endpoints; secure administrative endpoints. |
| **5** | **Stored XSS via Event Descriptions** | Medium | Medium | **Medium** | Apply strict input sanitization and context-aware output encoding on all user-submitted text. |
| **6** | **Denial of Service preventing Trip Registrations** | Medium | Low | **Low** | Implement rate-limiting, CAPTCHA, and deploy an Application Web Application Firewall (WAF). |