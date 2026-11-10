# Security Audit & Granular Remediation Deliverable (Nuclei)

**Student / Engineer:** Elias Alain (Eliasvit1)  
**Professor:** Alan Calvillo  
**Submission Date:** Day 14 (October 31, 2026)  
**Security Standard:** OWASP Top 10 / Dynamic Application Security Testing (DAST)  
**Tooling:** ProjectDiscovery Nuclei v3.11.1 (Templates v10.5.1)  

---

## 1. Executive Summary & Objective

This repository contains the consolidated deliverables and audit artifacts resulting from the comprehensive security assessment performed across the three multi-tier architecture repositories:
1. **Custom Dashboard** (Frontend): [`security-frontend`](https://github.com/Eliasvit1/security-frontend)
2. **Backend API**: [`security-api`](https://github.com/Eliasvit1/security-api)
3. **LDAP Authentication Service**: [`FastApi-Openldap`](https://github.com/Eliasvit1/FastApi-Openldap)

All audit reports, inline documentation, code fixes, and git commits have been implemented in **English** following a **strict granular commit approach** (one independent commit per remediated vulnerability).

---

## 2. Nuclei Execution Case: Docker Bridge Network (`ddos_labnet`)

To address the network connectivity and unreachable port consideration noted by the professor:
> *"si por algun motivo leen algo relacionado a que no puede acceder por el puerto. Posiblemente necesiten un bridge entre el contenedor de nuclei y lo que intentan analizar..."*

### Implemented Strategy (Case A: Docker Bridge Network):
* The official Nuclei container (`projectdiscovery/nuclei:latest`) was attached directly to the existing Docker bridge network (`ddos_labnet`) using:
  ```bash
  docker run --rm --network ddos_labnet \
    -v "C:\Users\Alain\Downloads\nuclei-audit-fixes\reports\<service>:/reports" \
    projectdiscovery/nuclei:latest \
    -target http://<service-container>:80 \
    -o /reports/nuclei-scan.txt \
    -je /reports/nuclei-scan.json -nc
  ```
* **Firewall / Fail2ban Handling:** The containerized defense mechanism (Fail2ban `http-flood` jail) detected scanner traffic and initially banned the scanner IP (`172.29.0.2`). To prevent false scanner lockout and port closure errors, the Docker bridge subnet (`172.29.0.0/16`) was whitelisted in Fail2ban (`fail2ban-client set http-flood addignoreip 172.29.0.0/16`).
* This enabled comprehensive DAST coverage across all ports and HTTP endpoints without dropping connections.

---

## 3. Vulnerability Findings & Granular Remediation Matrix

| Service | Finding ID | Severity | Vulnerability Description | Remediation Commit Hash & Message |
|---|---|---|---|---|
| **Custom Dashboard** | `VULN-FE-001` | Medium | Unsanitized username input leading to potential XSS / DOM injection | [`09e273e`](https://github.com/Eliasvit1/security-frontend/commit/09e273e) `fix(auth): sanitize user input to prevent XSS vulnerability` |
| **Custom Dashboard** | `VULN-FE-002` | Low | Missing defense-in-depth HTTP security headers & Nginx version disclosure | [`c2187d2`](https://github.com/Eliasvit1/security-frontend/commit/c2187d2) `fix(dashboard): enforce security headers and suppress server tokens` |
| **Backend API** | `VULN-BE-001` | Medium | Insecure CORS configuration (`allow_origins=["*"]` with credentials enabled) | [`e07f97b`](https://github.com/Eliasvit1/security-api/commit/e07f97b) `fix(api): restrict CORS origins and disable wildcard credentials` |
| **Backend API** | `VULN-BE-002` | Medium | Missing application-level rate limiting on sensitive `/api/data` endpoints | [`405f695`](https://github.com/Eliasvit1/security-api/commit/405f695) `fix(api): implement rate limiting on sensitive endpoints` |
| **Backend API** | `VULN-BE-003` | Low | Public exposure of interactive API docs (`/docs`, `/redoc`, `/openapi.json`) | [`3508863`](https://github.com/Eliasvit1/security-api/commit/3508863) `fix(api): disable unauthenticated OpenAPI and Swagger documentation endpoints` |
| **Backend API** | `VULN-BE-004` | Low | Missing security headers (`X-Frame-Options`, `CSP`, `nosniff`) & banner disclosure | [`831b6cc`](https://github.com/Eliasvit1/security-api/commit/831b6cc) `fix(api): enforce security headers and suppress server tokens` |
| **LDAP Service** | `VULN-LDAP-001` | High | OpenLDAP directory allows unauthenticated anonymous binds | [`e75e60a`](https://github.com/Eliasvit1/FastApi-Openldap/commit/e75e60a) `fix(ldap): disable anonymous binding and enforce authentication` |
| **LDAP Service** | `VULN-LDAP-002` | Medium | Lack of rate limiting on `/login` (credential stuffing / brute force risk) | [`afa2fe4`](https://github.com/Eliasvit1/FastApi-Openldap/commit/afa2fe4) `fix(auth): implement rate limiting on sensitive login endpoint` |
| **LDAP Service** | `VULN-LDAP-003` | Medium | Insecure CORS wildcard with credentials on auth endpoints | [`2ef22fc`](https://github.com/Eliasvit1/FastApi-Openldap/commit/2ef22fc) `fix(auth): restrict CORS origins and disable wildcard credentials` |
| **LDAP Service** | `VULN-LDAP-004` | Low | Public Swagger/OpenAPI exposure on authentication microservice | [`c93f39c`](https://github.com/Eliasvit1/FastApi-Openldap/commit/c93f39c) `fix(auth): disable unauthenticated OpenAPI and Swagger documentation endpoints` |
| **LDAP Service** | `VULN-LDAP-005` | Low | Missing security headers & server banner disclosure | [`36e6d0c`](https://github.com/Eliasvit1/FastApi-Openldap/commit/36e6d0c) `fix(auth): enforce security headers and suppress server tokens` |

---

## 4. GitHub Remote Repositories & Commit Logs

### 1. Custom Dashboard Repository (React)
* **Primary URL:** [https://github.com/Eliasvit1/security-dashboard-frontend.git](https://github.com/Eliasvit1/security-dashboard-frontend.git)
* **Mirror / Base URL:** [https://github.com/Eliasvit1/security-frontend.git](https://github.com/Eliasvit1/security-frontend.git)
* **Commit History:**
  * `42d1c7b` `refactor(frontend): migrate to React SPA with multi-page session persistence`
  * `c2187d2` `fix(dashboard): enforce security headers and suppress server tokens`
  * `09e273e` `fix(auth): sanitize user input to prevent XSS vulnerability`
  * `95341cb` `docs(security): add initial Nuclei security audit report`

### 2. Backend API Repository
* **Primary URL:** [https://github.com/Eliasvit1/security-dashboard-backend.git](https://github.com/Eliasvit1/security-dashboard-backend.git)
* **Mirror / Base URL:** [https://github.com/Eliasvit1/security-api.git](https://github.com/Eliasvit1/security-api.git)
* **Commit History:**
  * `4ca4bc6` `fix(cors): authorize React frontend development origins`
  * `831b6cc` `fix(api): enforce security headers and suppress server tokens`
  * `3508863` `fix(api): disable unauthenticated OpenAPI and Swagger documentation endpoints`
  * `405f695` `fix(api): implement rate limiting on sensitive endpoints`
  * `e07f97b` `fix(api): restrict CORS origins and disable wildcard credentials`
  * `70bb1b0` `docs(security): add initial Nuclei security audit report`

### 3. LDAP Service Repository
* **Remote URL:** [https://github.com/Eliasvit1/FastApi-Openldap.git](https://github.com/Eliasvit1/FastApi-Openldap.git)
* **Commit History:**
  * `31a907f` `fix(cors): authorize React frontend development origins`
  * `36e6d0c` `fix(auth): enforce security headers and suppress server tokens`
  * `c93f39c` `fix(auth): disable unauthenticated OpenAPI and Swagger documentation endpoints`
  * `2ef22fc` `fix(auth): restrict CORS origins and disable wildcard credentials`
  * `afa2fe4` `fix(auth): implement rate limiting on sensitive login endpoint`
  * `e75e60a` `fix(ldap): disable anonymous binding and enforce authentication`
  * `731a8b3` `docs(security): add initial Nuclei security audit report`

---

## 5. Verification & Post-Remediation Results

1. **Targeted Re-Scan (`-tags cors,swagger,openapi,redoc`):** Completed with **0 matches found**.
2. **HTTP Header Hardening:** `Server: nginx` (no version leakage), `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, and strict `Content-Security-Policy` verified active across all endpoints.
3. **Rate Limiter Validation:** Requests exceeding thresholds trigger `HTTP 429 Too Many Requests` with a valid `Retry-After` header.
4. **Anonymous Bind Verification:** Unauthenticated attempts to query OpenLDAP or login with empty credentials are rejected with `HTTP 400 Bad Request`.

---

## 6. Directory Structure of this Deliverable Package

```text
nuclei-audit-fixes/
├── README.md                              <- This executive document
├── ENTREGABLE_AUDITORIA_NUCLEI.md         <- Spanish submission guide for Professor Alan Calvillo
├── evidence/
│   ├── git-commits-history.txt            <- Full Git log hashes and commit metadata
│   └── service-test-results.txt           <- Runtime verification, headers, and rate limiting output
└── reports/
    ├── custom-dashboard/                  <- Raw Nuclei output, JSON, and audit report for Frontend
    │   ├── nuclei-scan.txt
    │   ├── nuclei-scan.json
    │   └── security-audit-report.md
    ├── backend-api/                       <- Raw Nuclei output, JSON, and audit report for Backend API
    │   ├── nuclei-scan.txt
    │   ├── nuclei-scan.json
    │   └── security-audit-report.md
    └── ldap-service/                      <- Raw Nuclei output, JSON, and audit report for LDAP
        ├── nuclei-scan.txt
        ├── nuclei-scan.json
        └── security-audit-report.md
```
