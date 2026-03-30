# VULN-004: Authentication Logic Weakness (Login Trust Boundary Failure)

## Summary
The login endpoint correctly rejects invalid credentials and does not
exhibit a standalone authentication bypass.

However, accounts created through improper registration validation
(VULN-001) can successfully authenticate without any additional
verification or restriction.

This indicates a trust boundary failure between the registration and
login components, resulting in a logical weakness in the authentication
design.

---

## Affected Endpoint
POST /rest/user/login

---

## Expected Security Model
- Only accounts created through fully validated registration flows
  should be eligible for authentication.
- Login logic should assume that persisted user objects conform to
  enforced credential and policy constraints.
- Registration and login should share a consistent validation contract
  or schema.

---

## Observed Behavior

### Successful Authentication
The following improperly registered accounts were able to authenticate
successfully:

- Accounts created with passwords shorter than the enforced UI policy
- Accounts created with mismatched `password` and `passwordRepeat`
  values

Authentication returned HTTP 200 OK along with valid JWT tokens.

### Failed Authentication
The following cases were correctly rejected:

- Accounts with empty email values
- Incorrect password submissions
- Missing password fields
- Blank (`" "`) password values

These attempts returned HTTP 401 Unauthorized.

---

## Root Cause Analysis
While the login endpoint performs correct credential matching, it
implicitly trusts that all stored user records meet required security
constraints.

Because registration allows invalid credential states (see VULN-001),
login becomes an enforcement point that lacks contextual awareness of
account integrity.

This represents a broken trust boundary between:
- User creation (registration)
- User authentication (login)

---

## Impact
- Accounts with weak or inconsistent credentials can be authenticated
  and used normally.
- Security policies enforced at the UI layer are effectively bypassed.
- In combination with automated registration abuse, this could lead to
  large numbers of low-integrity user accounts.

The impact of this issue is realized only when combined with VULN-001,
but the authentication logic independently fails to defend against
invalid account states.

---

## Relationship to Other Findings
- This issue depends on the existence of VULN-001
  (Authentication Registration Validation Bypass).
- VULN-004 does not constitute an independent authentication bypass,
  but rather confirms and amplifies the impact of VULN-001.

---

## OWASP / CWE Mapping
- OWASP Top 10 2021: A04 – Insecure Design
- CWE-287: Improper Authentication
- CWE-306: Missing Authentication for Critical Function (contextual)
