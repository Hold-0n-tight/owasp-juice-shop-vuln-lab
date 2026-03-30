## Product Search - Search Endpoint

**Endpoint**
- GET /rest/products/search?q=

**Description**
Product search endpoint was tested by intercepting requests via OWASP ZAP
and manipulating the `q` query parameter.

**Tested Payload**
- q: `'`
- q: `')--`

**Result**
- HTTP 500 Internal Server Error
- SQLite syntax error returned
- Raw SQL query disclosed in response body

**Impact**
- Error-based SQL Injection
- Disclosure of backend SQL query structure
- Potential for further injection-based attacks

**OWASP Mapping**
- A03: Injection
- A05: Security Misconfiguration
