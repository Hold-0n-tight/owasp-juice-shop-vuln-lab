# VULN-001: Authentication Validation Inconsistency (Register)

## 1. Scope
- Endpoint: POST /api/Users/
- Feature: User Registration
- Tooling: OWASP ZAP (Manual Request Replay)

---

## 2. Expected Behavior
The server should enforce the same validation rules as the client-side UI, including:
- Email must be non-empty and valid format
- Password must satisfy minimum length requirements
- password and passwordRepeat must match
- Invalid input should result in 4xx error responses

---

## 3. Actual Behavior
The server accepts malformed or invalid registration data and creates user accounts successfully.

Observed behaviors:
- Empty email ("") is accepted during registration
- Password length requirements are not enforced server-side
- password and passwordRepeat mismatch is ignored
- Server consistently responds with HTTP 201 Created

---

## 4. Test Cases and Results

### Case 1: Empty Email
Request:
- email: ""
- password: valid
- passwordRepeat: valid

Response:
- HTTP/1.1 201 Created
- User object created with empty email field

Login Result:
- Login fails ("Invalid email or password")

---

### Case 2: Short Password (Below UI Requirement)
Request:
- email: valid
- password: "qwe" (3 characters)
- passwordRepeat: "qwe"

Response:
- HTTP/1.1 201 Created
- User successfully created

Login Result:
- Login succeeds with weak password

---

### Case 3: Password Mismatch
Request:
- email: valid
- password: "qwe123"
- passwordRepeat: "qwe1234"

Response:
- HTTP/1.1 201 Created
- User successfully created

Login Result:
- Login succeeds despite mismatch

---

## 5. Root Cause Analysis
- Client-side validation exists but is not enforced on the server
- Registration and login flows apply inconsistent validation rules
- The backend implicitly trusts frontend validation

---

## 6. Security Impact
- Creation of accounts with weak or invalid credentials
- Authentication policy bypass
- Increased risk of:
  - Account enumeration
  - Credential stuffing
  - Inconsistent authentication state
- Violates secure-by-design principles

---

## 7. OWASP Mapping
- OWASP Top 10:
  - A2: Broken Authentication
  - A5: Security Misconfiguration

---

## 8. Notes for Further Investigation
- SecurityAnswer is created via a separate POST request
- Registration process is non-atomic
- Potential for further abuse in account recovery flows

---

## Validation Bypass Summary

| Case | Client-side Result | Server-side Result | Account Created | Login Possible |
|-----|-------------------|-------------------|-----------------|----------------|
| Normal | Allowed | 201 Created | Yes | Yes |
| Empty Email | Blocked | 201 Created | Yes | No |
| Short Password | Blocked | 201 Created | Yes | Yes |
| Password Mismatch | Blocked | 201 Created | Yes | Yes |

---

### Observation

During registration, the security answer is submitted via a separate
POST request to /api/SecurityAnswers/. This indicates a decoupled
persistence flow for credential recovery data.

This behavior was observed but not exploited at this stage.

