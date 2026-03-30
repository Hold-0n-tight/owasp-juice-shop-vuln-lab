# User Object – Observed Structure & Validation Flow

## Scope & Methodology

This document describes the **observed User object structure and its
validation lifecycle**, strictly based on:

- Direct HTTP requests issued during testing
- Actual server responses
- Observable system behavior

No assumptions are made about:
- Database schema
- ORM or framework internals
- Privileged access or administrative views

---

## Observed User Object Fields

Based on registration and login interactions, the following user-related
fields were **explicitly observed**.

### 1. email

- Supplied in the registration request body
- Accepted as an empty string (`""`)
- No server-side format or presence validation observed during registration
- Treated as an identifier during login

**Observation**
- Registration endpoint allows structurally invalid email values
- Login endpoint assumes email validity

---

### 2. password

- Supplied in plaintext during registration and login
- Server does not enforce:
  - Minimum length
  - Password complexity
- `password` and `passwordRepeat` mismatch is accepted during registration

**Observation**
- Password policy is inconsistently enforced across lifecycle stages

---

### 3. passwordRepeat

- Present only during registration
- Not validated against `password` server-side
- Ignored post-registration

**Observation**
- Used only as a client-side convenience field

---

### 4. User Identifier (id)

- Assigned by the server upon successful registration
- Used implicitly during authentication

**Observation**
- User record is persisted regardless of credential integrity

---

## Validation Lifecycle Analysis

The following table summarizes **where validation is enforced and where it is not**.

| Field | Registration | Login | Notes |
|------|-------------|-------|------|
| email | ❌ Not validated | ✅ Assumed valid | Trust boundary mismatch |
| password | ❌ Policy not enforced | ✅ Compared | Weak credential persistence |
| passwordRepeat | ❌ Not checked | N/A | Client-side only |
| user existence | ✅ Created | ✅ Used | No integrity re-check |

---

## Authentication Logic Assumptions (Derived from Behavior)

From the observed behavior, the system implicitly assumes:

1. Any user record created via the registration endpoint is valid by default.
2. The login process does not re-validate the structural integrity of user credentials.
3. Credential correctness is evaluated without verifying credential quality.

These assumptions introduce **logical inconsistencies** between user creation
and user authentication.

---

## Security Implications

- User records with malformed or weak credentials can persist in the system.
- Authentication logic relies on data that was never server-validated.
- This creates a foundation for **authentication logic vulnerabilities**
  without requiring:
  - Database access
  - Administrative privileges
  - Exploitation of memory or injection flaws

---

## Relation to Identified Vulnerabilities

- **VULN-001**: Registration validation bypass
- **VULN-004**: Authentication logic trust inconsistency (candidate)

This document serves as a **design-level attack surface reference**
for subsequent vulnerability analysis.
