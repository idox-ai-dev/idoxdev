# idoxdev
# Sample request/response for workflows

## Sanitize before submitting

**Workflow:** User has text that may contain PII/PHI. Before pasting into OpenAPI try-it (or another tool), they (1) run PII extraction, (2) redact using the result, (3) submit only the redacted text.

| File | Use |
|------|-----|
| `sanitize-before-submit-request.json` | Full request: `request` (method, url, headers) and `body`. Use with curl + jq (see below) or as reference. Replace `YOUR_API_KEY` in the file or override in the command. |
| `sanitize-before-submit-body.json` | Request body only. Use directly with curl: `curl ... -d @sanitize-before-submit-body.json`. |
| `sanitize-before-submit-response.json` | Example response. Use `items[].entities[].offset` and `length` (or `text`) to replace spans in the original paragraph with `[REDACTED]` or by category (e.g. `[REDACTED-Person]`). |

**Step 3 (client-side):** Build redacted text from the original `paragraph` and entity `offset`/`length`; then paste or POST that string into your target (OpenAPI, form, etc.).

---

## How to make the API call

### cURL using the request JSON directly (with jq)

Read URL, headers, and body from `sanitize-before-submit-request.json`. Set your API key via env or replace `YOUR_API_KEY` in the file first.

```bash
# From repo root; requires jq
FILE="docs/samples/sanitize-before-submit-request.json"
curl -s -X POST "$(jq -r '.request.url' "$FILE")" \
  -H "iDox-API-Engine-Key: ${IDOX_API_KEY:-YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "User-Agent: $(jq -r '.request.headers["User-Agent"]' "$FILE")" \
  -d "$(jq -c '.body' "$FILE")"
```

Example with inline key: `IDOX_API_KEY=your_key_here bash -c '...'` or edit the JSON and use `YOUR_API_KEY` in the file.

### cURL with body file only

If you only need the body as a file (URL and headers you pass yourself):

```bash
curl -X POST "https://docs.idox.ai/api/v3/ner/pii" \
  -H "iDox-API-Engine-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d @docs/samples/sanitize-before-submit-body.json
```

Replace `YOUR_API_KEY` with your real API key.

### cURL (inline body)

```bash
curl -X POST "https://docs.idox.ai/api/v3/ner/pii" \
  -H "iDox-API-Engine-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "User-Agent: YourApp/1.0" \
  -d '{
    "paragraphs": [
      "Patient John Smith (DOB 1985-03-15) visited the clinic on 2024-01-10. Contact: j.smith@email.com, phone 555-123-4567. SSN: 123-45-6789. Address: 456 Oak St, Boston, MA 02101."
    ],
    "region": "US",
    "language": "English",
    "confidenceScore": 0.5,
    "categoriesFilter": []
  }'
```

Replace `YOUR_API_KEY` with your real API key.

### Python (requests)

```python
import requests

url = "https://docs.idox.ai/api/v3/ner/pii"
headers = {
    "iDox-API-Engine-Key": "YOUR_API_KEY",
    "Content-Type": "application/json",
    "User-Agent": "YourApp/1.0",
}
body = {
    "paragraphs": [
        "Patient John Smith (DOB 1985-03-15) visited the clinic on 2024-01-10. Contact: j.smith@email.com, phone 555-123-4567. SSN: 123-45-6789. Address: 456 Oak St, Boston, MA 02101."
    ],
    "region": "US",
    "language": "English",
    "confidenceScore": 0.5,
    "categoriesFilter": [],
}

response = requests.post(url, json=body, headers=headers)
print(response.status_code)
print(response.json())
```

### Using the JSON file (Python)

To read the sample file and send the request:

```python
import json
import requests

with open("docs/samples/sanitize-before-submit-request.json") as f:
    data = json.load(f)

# Replace placeholder key if needed
headers = {**data["request"]["headers"], "iDox-API-Engine-Key": "YOUR_API_KEY"}
response = requests.post(
    data["request"]["url"],
    json=data["body"],
    headers=headers,
)
print(response.json())
```

