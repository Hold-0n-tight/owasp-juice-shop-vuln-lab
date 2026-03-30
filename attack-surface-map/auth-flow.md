# Authentication Flow Map

## 1. Purpose & Scope

This document models the **authentication trust flow** of the Juice Shop application.
Rather than listing endpoints only, it focuses on:

- Where validation decisions are made
- Which components trust which prior states
- How invalid states propagate across authentication boundaries

This flow analysis explains the structural root cause behind:
- **VULN-001 (Registration Validation Bypass)**
- **VULN-004 (Authentication Logic Inconsistency)**

---

## 2. Trust Boundary Overview

'''
[ Client ] ──(untrusted)──▶ [ Server API ] ──(trusted)──▶ [ Auth Logic ]
'''


**Critical Observation**

The server implicitly trusts client-enforced constraints during registration,
but later assumes strong invariants during login.

This mismatch creates inconsistent User states.

---

## 3. High-Level Authentication Lifecycle

'''
[Client UI]
│
├─ (1) Registration Input Validation
│ - Enforced ONLY in client
│
├─ (2) POST /api/Users/
│ - User persisted
│ - ❌ No server-side invariant enforcement
│
├─ (3) POST /api/SecurityAnswers/
│ - Executed independently
│ - Weak transactional coupling
│
└─ (4) POST /rest/user/login
- Assumes valid User state
- Issues JWT
'''


---

## 4. Registration Phase Analysis

### 4.1 Client-Side Validation (Non-Authoritative)

Enforced rules:
- Email format
- Password length
- Password repeat match

These checks are advisory and can be bypassed.

Client-side validation must never define system invariants,
yet in this flow, it effectively does.

---

### 4.2 Server-Side User Creation (POST /api/Users/)

Observed behavior:

- Accepts empty email values
- Accepts passwords shorter than defined policy
- Does not validate password vs passwordRepeat consistency

Result:

User records are persisted in invalid or inconsistent states.
These invalid states are later treated as trusted authentication objects.

This is the root defect, not merely a validation bug.

---

### 4.3 Security Answer Handling (POST /api/SecurityAnswers/)

Characteristics:

- Executed as a separate API call after user creation
- Not transactionally coupled with `/api/Users/`

Implications:

- User creation succeeds even if security answer logic were to fail
- Authentication-related data is fragmented across weakly coupled transactions

---

## 5. Login Phase Analysis

### 5.1 Login Assumptions (POST /rest/user/login)

Login logic assumes:

- Email is non-empty and well-formed
- Password adheres to policy
- Stored credential state is internally consistent

These assumptions are not guaranteed by the registration process.

---

### 5.2 Empirical Outcomes

| User State Origin | Registration Allowed | Login Allowed |
|------------------|----------------------|---------------|
| Empty email      | Yes                  | No            |
| Short password   | Yes                  | Yes           |
| Password mismatch| Yes                  | Yes           |

This demonstrates validation asymmetry between authentication stages.

---

## 6. Core Structural Insight

Validation rules are not consistent across authentication boundaries.

Registration is permissive, while login is selective.

This violates the principle of end-to-end input validation and results in:

- Weak-credential accounts that fully authenticate
- Orphaned or unusable accounts
- Unpredictable authentication states

---

## 7. Related Vulnerabilities

- **VULN-001**: Authentication Registration Validation Bypass
- **VULN-004**: Authentication Logic Inconsistency (derived impact)

---

## 8. Remediation (Conceptual)

### 8.1 Enforce Server-Side Authentication Invariants

All authentication-related invariants must be enforced exclusively on the server,
including:

- Email presence and format validation
- Password length and complexity requirements
- Password and passwordRepeat consistency checks

Client-side validation should be treated as UX enhancement only.

---

### 8.2 Define a Single Source of Truth for User Validity

The system must define and enforce a clear definition of a valid User object
at creation time.

Invalid user states should never be persisted as trusted authentication entities.

---

### 8.3 Align Registration and Login Assumptions

Login logic must never assume that persisted user records are valid by default.

Either:
- Registration must strictly enforce all invariants, or
- Login must re-validate invariant compliance

---

### 8.4 Strengthen Transactional Integrity

Authentication-related operations (user creation, security answers)
should be atomically executed or safely rolled back on failure.

This prevents partially valid identities from persisting.
