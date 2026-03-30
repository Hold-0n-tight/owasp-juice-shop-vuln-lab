# VULN-002: Product Search API Input Handling Weakness

## Summary

The product search functionality exposes an API endpoint that accepts user-controlled query parameters (`q`) without clear validation or normalization.  
Multiple abnormal input patterns were accepted by the backend, resulting in inconsistent responses and revealing insufficient input handling at the API layer.

This behavior indicates a weak trust boundary between client input and server-side query processing.

---

## Affected Endpoint

'''
GET /rest/products/search?q=<user_input>
'''


---

## Attack Surface Description

The search feature is accessible via:

- UI search input field
- Direct URL manipulation of the `q` query parameter

Observed behavior suggests that the frontend UI does not always directly reflect the actual request sent to the backend, indicating potential client-side abstraction or caching.

---

## Input Variations Tested

The following input cases were manually tested by directly manipulating the request in an intercepting proxy:

- Case A: `q` parameter removed
- Case B: `q` parameter present but empty (`q=`)
- Case C: `q` parameter structured as an array-like value
- Case D: Multiple `q` parameters in a single request
- Case E: Unexpected data types (numeric, special characters, symbols)

---

## Observed Behavior

- The backend accepted malformed or unexpected `q` values without explicit rejection.
- Responses frequently returned `304 Not Modified`, suggesting:
  - Insufficient differentiation of request states
  - Possible overreliance on caching mechanisms
- The server did not return validation errors or input-related error messages.
- The same response structure was returned even when the semantic meaning of the query changed.

---

## Security Impact

While no direct data exfiltration was observed at this stage, the behavior introduces the following risks:

- Increased attack surface for injection-style vulnerabilities
- Undefined backend behavior for unexpected input types
- Reduced reliability of server-side assumptions about input structure
- Potential bypass of business logic dependent on search filtering

---

## Root Cause Hypothesis

The backend appears to:

- Trust the existence and structure of the `q` parameter
- Perform minimal or no normalization of user input
- Fail to enforce strict input schemas at the API boundary

---

## Recommended Remediation

- Enforce strict input validation for the `q` parameter
  - Reject unexpected data types
  - Enforce length and character constraints
- Normalize query input before processing
- Return explicit client error responses (4xx) for invalid input
- Review caching logic to ensure request variations are properly differentiated

---

## Notes

Detailed experimental observations and raw request/response data are documented in `notes.md`.
