## Authentication - Login Endpoint

**Endpoint**
- POST /rest/user/login

**Description**
Login endpoint was tested by bypassing frontend validation using ZAP proxy.

**Tested Payload**
- email: `'`
- password: `test`

**Result**
- HTTP 500 Internal Server Error
- SQLite error message disclosed
- Raw SQL query exposed

**Impact**
- Error-based SQL Injection
- Internal database structure disclosure

**OWASP Mapping**
- A03: Injection
- A05: Security Misconfiguration
