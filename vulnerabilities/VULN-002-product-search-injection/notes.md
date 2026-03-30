# VULN-002 Investigation Notes

## Initial Observation

When entering search terms via the UI (e.g., "banana"), the expected query value did not consistently appear in intercepted requests.

Observed request:
'''
GET /rest/products/search?q=
'''

Despite user input, the `q` parameter was sometimes empty, while the UI continued to display filtered results.

---

## Hypothesis

- Products may be preloaded on page load
- Client-side filtering may be applied after initial data retrieval
- Search input may not always trigger a new backend query

This required validation via direct API interaction rather than UI-driven testing.

---

## Methodology

- Intercepted requests using ZAP
- Manually modified and replayed requests
- Observed server response codes, headers, and caching behavior
- Compared responses across different malformed inputs

---

## Key Findings

- Removing or altering the `q` parameter did not trigger validation errors
- Multiple malformed inputs resulted in identical responses
- Frequent `304 Not Modified` responses suggest aggressive caching
- Backend did not signal invalid input conditions

---

## Additional Observations

- A separate request to `/api/Quantitys/` consistently accompanied search requests
- The purpose of this endpoint appears unrelated to query filtering
- The presence of this request may obscure analysis if not filtered out

---

## Open Questions

- Is query handling logic shared with other endpoints?
- Is search filtering performed server-side or client-side?
- Are there hidden parameters affecting search behavior?

These questions remain outside the scope of this vulnerability but indicate areas for further review.
