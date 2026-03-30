# VULN-001: Authentication Registration Validation Bypass

## Summary
During user registration, client-side validation is enforced for
email format and password policies. However, the server does not
properly validate these constraints.

As a result, accounts can be created with invalid or inconsistent
credentials by directly manipulating HTTP requests.

## Affected Endpoint
POST /api/Users/

## Observed Behavior
- Empty email values are accepted by the server
- Password length policies are not enforced server-side
- Password and passwordRepeat mismatch is not validated

## Impact
This vulnerability allows attackers to create accounts with weak or
unexpected credentials, potentially enabling account abuse and
inconsistent authentication behavior.
This may also result in undefined authentication behavior,
as accounts with inconsistent credentials can still successfully log in.

## Evidence
See the evidence directory for captured HTTP requests and responses.

## OWASP Mapping
- OWASP Top 10 2021: A04 – Insecure Design
- CWE-20: Improper Input Validation

___________________________

## Remediation

### 1. Enforce Server-Side Authentication Invariants at Registration

All authentication-related invariants **must be enforced on the server at the time of user creation**.
Client-side validation must be treated strictly as a UX convenience and never as a security control.

At minimum, the server **MUST** validate:

- Email presence (non-empty)
- Email format (RFC-compliant or equivalent)
- Password minimum length and policy
- Password and passwordRepeat consistency

Invalid user states must be rejected before persistence.

---

### 2. Prevent Persistence of Invalid Authentication States

The root cause of this vulnerability is not merely weak validation,
but the **persistence of logically invalid User objects**.

The system should define a clear invariant for a “valid User”:

- A User object that does not satisfy authentication invariants
  **must never be written to the database**.
- Partial or malformed user records should be rejected or rolled back.

Persisting invalid authentication entities creates downstream inconsistencies
that cannot be safely corrected at login time.

---

### 3. Align Registration and Login Assumptions

Currently, registration and login operate under different assumptions
about what constitutes a valid User state.

This asymmetry must be eliminated.

Either:

- Registration strictly enforces all invariants, allowing login logic
  to safely trust persisted User records

or

- Login logic explicitly re-validates all authentication invariants
  before issuing tokens

The former is strongly recommended.

---

### 4. Strengthen Transactional Integrity of Identity Creation

Authentication-related operations are currently split across multiple API calls
(e.g., user creation and security answer submission).

To prevent partially valid identities:

- User creation and associated authentication metadata
  should be executed atomically
- Failures in any authentication-critical step should trigger rollback
- A User object should not become login-eligible until all required
  identity components are successfully completed

---

### 5. Security Impact of Proper Remediation

Applying the above remediations will:

- Eliminate weak-credential accounts created via client-side bypass
- Remove orphaned or unusable user states
- Ensure consistent authentication behavior across all flows
- Reduce the attack surface for logic-based authentication vulnerabilities

This remediation directly addresses both:

- **VULN-001 (Registration Validation Bypass)**, and
- **VULN-004 (Authentication Logic Inconsistency)**

by restoring end-to-end trust consistency in the authentication lifecycle.

