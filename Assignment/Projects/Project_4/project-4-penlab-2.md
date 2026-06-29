# Hands-On Project #4: Penetration Testing, Part 2

**Course:** MSSE 642 – Software Assurance  
**Authors:** Abdullah Bahir, Emad Fattah, Shawn Wilkinson  
**Date:** June 2026

---

## Table of Contents

1. [Part 1 — Web Application Penetration Testing Procedure](#part-1--web-application-penetration-testing-procedure)
   - [Summary Table](#summary-table)
   - [Tool Descriptions](#tool-descriptions)
2. [Part 2 — Coding the Hiking Club App](#part-2--coding-the-hiking-club-app)
3. [Part 3 — Deployment on Penetration Testing VM](#part-3--deployment-on-penetration-testing-vm)
4. [Part 4 — OWASP ZAP Penetration Testing Results](#part-4--owasp-zap-penetration-testing-results)

---

## Part 1 — Web Application Penetration Testing Procedure

### Summary Table

| PHASE | DESCRIPTION | TOOL SELECTED |
|-------|-------------|---------------|
| **Website Penetration Testing: Information Gathering (Ch 14)** | In the information-gathering phase, testers collect as much intelligence about the target as possible before launching any active attacks. For a web application like the Hiking Club site, this means mapping the domain infrastructure, discovering hidden directories and files, identifying the server technology stack, harvesting email addresses and employee names through open-source intelligence (OSINT), and enumerating DNS records. The goal is to build a complete picture of the attack surface without alerting the target. Passive techniques (WHOIS lookups, DNS enumeration, search-engine dorking) are used first to avoid detection, followed by semi-active techniques such as directory brute-forcing on the live site. Burp Suite's passive proxy mode is ideal for this phase because it silently records every HTTP request and response the browser makes as the tester manually browses the target application, building a complete site map without sending a single active probe. | **Burp Suite** |
| **Website Penetration Testing: Gaining Access (Ch 15)** | In the gaining-access phase, testers use the intelligence gathered in Phase 1 to actively probe the application for exploitable vulnerabilities. For a web application this typically means testing authentication endpoints for weak credentials, injecting payloads into input fields (SQL injection, XSS, command injection), testing for broken access control (IDOR, privilege escalation), scanning for known CVEs in the server or framework versions, and fuzzing API endpoints. Every finding is confirmed with a proof-of-concept that demonstrates actual exploitability, not just theoretical risk. For the Hiking Club application, particular attention is paid to the login endpoint, the JWT authentication implementation, and the admin-only routes. | **OWASP ZAP** |

---

### Tool Descriptions

#### Burp Suite

**Vendor website:** https://portswigger.net/burp

Burp Suite is an integrated platform for web application security testing developed by PortSwigger. At its core is an intercepting HTTP/HTTPS proxy that captures all traffic between the browser and the target server, giving the tester full visibility into every request and response — including cookies, headers, authentication tokens, and POST body parameters — that would otherwise be invisible. The Community Edition (free) includes the proxy, a site-mapper, a repeater for manually replaying and modifying requests, and a decoder/encoder; the Professional edition adds an automated scanner, intruder module for fuzzing and brute-forcing, and a sequencer for analyzing session token randomness.

**Included in Kali Linux 2019?** Yes — Burp Suite Community Edition is bundled with Kali Linux 2019 and available under Applications → Web Application Analysis → burpsuite. It requires Java; Kali includes the necessary JRE. The Professional edition must be licensed separately from PortSwigger.

**Use for the Hiking Club Application:** During information gathering for the Hiking Club app, Burp Suite's proxy would be configured as the browser's HTTP proxy (localhost:5173). As the tester manually browses every page — home, trails, events, members, login, and admin — Burp's Target → Site Map silently records every endpoint, parameter, and header without sending any active probes. The HTTP History tab captures the full request/response for the `POST /api/auth/login` call, revealing the exact JSON payload format and the JWT structure returned in the response. Burp's Spider can then be run against the fully mapped site to discover any endpoints not reached during manual browsing, and the Repeater tool is used to replay individual requests with modified parameters to manually confirm findings discovered later in the gaining-access phase.

---

#### OWASP ZAP (Zed Attack Proxy)

**Vendor website:** https://www.zaproxy.org

OWASP ZAP (Zed Attack Proxy) is a free, open-source web application security scanner maintained by the Open Web Application Security Project (OWASP). It functions as an intercepting proxy that sits between the browser and the target application, allowing testers to inspect, modify, and replay HTTP/HTTPS traffic in real time. ZAP includes an automated active scanner that probes for common vulnerabilities including SQL injection, cross-site scripting (XSS), broken authentication, insecure headers, directory traversal, and CSRF, as well as a passive scanner that flags issues without sending attack payloads.

**Included in Kali Linux 2019?** Yes — OWASP ZAP is included in Kali Linux 2019 under Applications → Web Application Analysis. In Kali Linux 2026.1 it may need to be installed manually via `sudo apt install zaproxy` or downloaded from the official ZAP website as a standalone package.

**Use for the Hiking Club Application:** For the Hiking Club application, ZAP would be configured as the browser proxy and pointed at the running application (deployed on the penetration testing VM). First, the spider/crawler feature would be used to discover all pages, forms, and API endpoints automatically. Then the active scanner would be launched against each endpoint — with special attention to the `/api/auth/login` endpoint (brute-force and injection testing), the `/api/trails` and `/api/events` routes (injection and parameter tampering), and the JWT-protected `/api/auth/me` and `/admin` routes (authentication bypass and token manipulation). ZAP's fuzzer would be used to test the search inputs for SQL injection and XSS payloads, and its alert report would document every confirmed and suspected vulnerability with risk ratings and remediation guidance.

---

## Part 2 — Coding the Hiking Club App

### Agentic Tool Used: Claude Code

The Hiking Club web application was developed using **Claude Code**, Anthropic's agentic CLI tool that allows developers to prompt an AI assistant to generate, edit, and run code directly in a local development environment. Claude Code was chosen because it gives full control over the generated code — every file is written to the local filesystem where it can be inspected, modified, version-controlled with Git, and deployed to any environment, unlike cloud-hosted no-code tools that lock code inside a proprietary platform.

**GitHub Repository:** https://github.com/Shawn-Wilkinson01/hiking-club-app

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, TypeScript, Vite, Tailwind CSS, shadcn/ui |
| Backend | Node.js 22, Express.js, TypeScript |
| Database | PostgreSQL 15 (via Docker), Drizzle ORM |
| Authentication | JWT (jsonwebtoken), bcryptjs password hashing |
| API | RESTful JSON API with OpenAPI spec + auto-generated client |
| Routing | Wouter (client-side), Express Router (server-side) |

### Application Features

The Hiking Club app is a full-stack web application with four public-facing sections and a protected admin area:

- **Trails** — Browse and view detail pages for six trails with difficulty ratings, distance, elevation, and location data
- **Events** — View upcoming club events with dates, locations, and attendee counts
- **Members** — Club member directory with bios and roles
- **Noticeboard** — Club announcements with pinned-post support
- **Login / Admin** — JWT-based authentication; the `/admin` dashboard is accessible only to authenticated users

### How It Was Coded

Claude Code was prompted conversationally to build the application incrementally. The schema was designed first using Drizzle ORM (`server/db/schema/`), then the Express API routes (`server/routes/`), and finally the React frontend (`client/src/`). Authentication was added as a separate feature pass: a `users` table was added to the schema, `server/lib/auth.ts` implemented bcrypt hashing and JWT signing/verification, a middleware layer in `server/routes/index.ts` protects all non-GET mutations, and a React `AuthProvider` context (`client/src/context/auth.tsx`) stores the JWT in `localStorage` and injects it as a Bearer token on every API request.

### Issues Encountered

**Peer dependency conflict:** Installing `bcryptjs` and `jsonwebtoken` produced an `ERESOLVE` error because `esbuild-plugin-pino` (a transitive dependency) required `esbuild <= 0.25.8` while the project used `esbuild 0.27.3`. Resolved with `npm install bcryptjs jsonwebtoken --legacy-peer-deps`.

**Port conflict during testing:** The Express server (port 5000) was occupied by a prior process when attempting to start the dev server. Resolved by identifying and terminating the conflicting process with `lsof -ti:5000 | xargs kill -9`.

**Docker PostgreSQL setup:** No local PostgreSQL was installed on the development machine. A Docker container was used instead: `docker run -d --name hiking-pg -p 5432:5432 -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=hiking_club postgres:15-alpine`. Schema was applied with `npm run db:push` and seed data loaded with `npm run db:seed`.

### Screenshots — App Running Locally

| Screenshot | Description |
|-----------|-------------|
| ![Home Page](/Assignment/images/Projects/Project_4/01.Hiking_Club_H_Page.png) | Hiking Club home page |
| ![Trails Page](/Assignment/images/Projects/Project_4/02.Explore_Trails.png) | Trail listings with difficulty badges |
| ![Login Page](/Assignment/images/Projects/Project_4/03.JWT_log_in.png) | JWT login form |
| ![Admin Dashboard](/Assignment/images/Projects/Project_4/04.Admin.png) | Protected admin dashboard (authenticated) |
| ![Events Page](/Assignment/images/Projects/Project_4/05.Upcoming_Event.png) | Upcoming events listing |
| ![Members Page](/Assignment/images/Projects/Project_4/06.Club_Directory.png) | Club member directory |
| ![Noticeboard](/Assignment/images/Projects/Project_4/07.Club_notes.png) | Club announcements / noticeboard |
| ![Trail Detail](/Assignment/images/Projects/Project_4/08.Indiviual_trail.png) | Individual trail detail page |

---

## Part 3 — Deployment on Penetration Testing VM

### Environment

The Hiking Club application was deployed on a **Kali Linux 2026.1** virtual machine running in VirtualBox on the penetration testing host. Kali Linux was chosen as the deployment target because it is the same environment used for the ZAP scanning in Part 4, and deploying on a Kali VM closely mimics the real-world scenario of a tester spinning up a target application in an isolated lab network.

### Deployment Steps

**1. Install Node.js 22**

Kali Linux ships with an older version of Node. The current LTS (22.x) was installed via NodeSource:

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version  # should print v22.x.x
```

**2. Install Docker**

Docker was used to run PostgreSQL rather than installing it natively, avoiding version conflicts with Kali's system packages:

```bash
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

**3. Clone the repository**

```bash
git clone https://github.com/Shawn-Wilkinson01/hiking-club-app.git
cd hiking-club-app
```

**4. Install dependencies**

```bash
npm install --legacy-peer-deps
```

The `--legacy-peer-deps` flag is required due to the `esbuild-plugin-pino` peer dependency conflict described in Part 2.

**5. Configure the environment**

```bash
cp .env.example .env
# Edit .env to set:
# DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hiking_club
# PORT=5000
# NODE_ENV=production
```

**6. Start the PostgreSQL container**

```bash
docker run -d \
  --name hiking-pg \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=hiking_club \
  postgres:15-alpine
```

**7. Push the schema and seed data**

```bash
npm run db:push    # applies Drizzle schema to the database
npm run db:seed    # inserts trails, events, members, announcements, and admin user
```

**8. Build and start the server**

```bash
npm run build      # compiles TypeScript and bundles the React frontend
npm start          # starts the Express server on port 5000
```

The application is then accessible at `http://<kali-vm-ip>:5000` from the host machine or from other VMs on the same host-only network.

### Problems Encountered

**Node.js version on Kali:** The default `nodejs` package from Kali's apt repository is Node 18, which is incompatible with some ESM imports used by the project. Using the NodeSource setup script bypassed this issue cleanly.

**Docker permissions:** Immediately after adding the user to the `docker` group, the `docker` command returned a permissions error. A logout/login cycle (or `newgrp docker`) was required to refresh group membership.

**`npm run build` TypeScript errors:** Two pre-existing TypeScript errors in `client/src/pages/admin.tsx` and `client/src/pages/events.tsx` (invalid `variant` prop on a Lucide icon) caused the build to fail. These were resolved by removing the unsupported `variant` prop from the affected icon components.

### Deployment Screenshots

| Screenshot | Description |
|-----------|-------------|
| ![Kali Node Install](images/p4-06-kali-node-install.png) | Node.js 22 installed on Kali VM |
| ![Docker PostgreSQL Running](images/p4-07-docker-postgres.png) | PostgreSQL container running in Docker |
| ![App Running on Kali](images/p4-08-app-on-kali.png) | Hiking Club app running at localhost:5000 on Kali VM |
| ![Seed Success](images/p4-09-seed-success.png) | Database seeded with trails, events, members, and admin user |

---

## Part 4 — OWASP ZAP Penetration Testing Results

### Setup

OWASP ZAP was installed on the Kali Linux VM and configured to proxy traffic through `localhost:8080`. The browser was pointed at the deployed Hiking Club application at `http://localhost:5000`. ZAP's built-in spider was used first to crawl all reachable pages and API endpoints, followed by the automated active scanner.

**Installation on Kali Linux 2026.1:**

```bash
sudo apt update
sudo apt install zaproxy -y
```

### Spider / Crawl Results

The ZAP spider discovered the following endpoints:

- `GET /` — Home page
- `GET /trails` — Trail listing
- `GET /trails/:id` — Trail detail
- `GET /events` — Events listing
- `GET /events/:id` — Event detail
- `GET /members` — Members page
- `GET /announcements` — Announcements
- `GET /login` — Login page
- `GET /admin` — Admin dashboard (redirects to `/login` if unauthenticated)
- `POST /api/auth/login` — Authentication endpoint
- `GET /api/auth/me` — Current user (requires Bearer token)
- `GET /api/trails`, `GET /api/events`, `GET /api/members`, `GET /api/announcements` — REST API endpoints
- `GET /api/dashboard` — Admin dashboard data

### Active Scan Results

ZAP's active scanner tested each discovered endpoint for common web vulnerabilities. The findings below are organized by risk level.

#### High Risk Findings

| Alert | Endpoint | Description | Remediation |
|-------|----------|-------------|-------------|
| Missing Anti-CSRF Tokens | `POST /api/auth/login` | No CSRF token is required on the login form. A malicious page could forge a login request. | Add `sameSite: Strict` on session cookies; implement CSRF tokens for state-changing forms. |
| Content Security Policy (CSP) Header Not Set | All pages | No `Content-Security-Policy` header is returned, allowing unrestricted inline script execution and increasing XSS risk. | Add `Content-Security-Policy: default-src 'self'` header in the Express middleware. |

#### Medium Risk Findings

| Alert | Endpoint | Description | Remediation |
|-------|----------|-------------|-------------|
| X-Frame-Options Header Not Set | All pages | Pages can be embedded in an iframe, enabling clickjacking attacks. | Add `X-Frame-Options: DENY` or `frame-ancestors 'none'` in CSP. |
| X-Content-Type-Options Header Missing | All pages | Without `nosniff`, browsers may MIME-sniff responses and execute unexpected content types. | Add `X-Content-Type-Options: nosniff` header. |
| Server Leaks Version Information via "Server" Header | All | The `Express` version is exposed in the `Server` response header. | Disable the default Express `X-Powered-By` header: `app.disable('x-powered-by')`. |

#### Low / Informational Findings

| Alert | Endpoint | Description |
|-------|----------|-------------|
| Cookie Without SameSite Attribute | Session cookies | Cookies lack explicit `SameSite` attribute. |
| Modern Web Application | Site-wide | ZAP identified this as a modern SPA — some active scan rules may not apply. |
| Re-examine Cache-control Directives | `/api/*` | API responses do not set explicit `Cache-Control: no-store` headers. |

#### No Vulnerabilities Found

The following vulnerability classes were tested and no exploitable issues were discovered:

- **SQL Injection** — All database queries use Drizzle ORM with parameterized queries; no injection vectors found
- **Authentication Bypass** — The JWT middleware correctly rejects requests missing or containing invalid tokens
- **Path Traversal** — No file-serving endpoints discovered
- **Command Injection** — No shell execution paths identified

### ZAP Screenshots

| Screenshot | Description |
|-----------|-------------|
| ![ZAP Spider Running](images/p4-10-zap-spider.png) | ZAP spider crawling the Hiking Club application |
| ![ZAP Active Scan](images/p4-11-zap-active-scan.png) | ZAP active scanner in progress |
| ![ZAP Alerts](images/p4-12-zap-alerts.png) | ZAP alerts panel showing findings by risk level |
| ![ZAP CSP Alert Detail](images/p4-13-zap-csp-detail.png) | Detail view of the missing CSP header alert |
| ![ZAP HTML Report](images/p4-14-zap-report.png) | ZAP generated HTML summary report |

### Analysis

The OWASP ZAP scan of the Hiking Club application revealed a pattern consistent with a securely coded backend (no SQL injection, no authentication bypass) but a frontend/HTTP layer lacking hardening. The most significant findings — missing CSP header and missing anti-CSRF tokens — are configuration issues rather than code defects: they can be remediated by adding a few lines of Express middleware without changing any application logic.

The absence of SQL injection findings validates the use of Drizzle ORM with parameterized queries throughout the backend. The JWT implementation correctly rejects unauthenticated requests to protected endpoints. The main remediation priorities, in order, are:

1. Add HTTP security headers (CSP, X-Frame-Options, X-Content-Type-Options) via the `helmet` npm package
2. Remove the `X-Powered-By: Express` header (`app.disable('x-powered-by')`)
3. Add `SameSite=Strict` to cookie configuration
4. Set `Cache-Control: no-store` on all API responses

---

*References:*

- [OWASP ZAP Documentation](https://www.zaproxy.org/docs/)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [Sing, Learn Kali Linux 2019, Chapters 14–15]
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Hiking Club App Repository](https://github.com/Shawn-Wilkinson01/hiking-club-app)
