# Interpret and return HTTP status codes correctly

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @medbouhezza — 2026-10-14 — https://www.instagram.com/reel/DeehBXKERyM/

### On-screen items from sampled frames
- `2xx`: success; example `200 OK` and `201 Created`.
- `3xx`: redirection; `301 Moved Permanently` appears in the clip.
- `4xx`: client-side request problems; `401 Unauthorized` (authentication needed), `403 Forbidden`, and `404 Not Found` are illustrated.
- The clip contrasts a `404` response with a successful `200` response, and recommends monitoring the resulting status codes rather than hiding errors.

### Implementation guidance (synthesis)
- `400`: invalid request syntax/parameters; `401`: unauthenticated; `403`: authenticated but not permitted; `404`: absent/hidden resource; `409`: state conflict; `422`: semantically invalid content when your API uses that convention; `429`: too many requests.
- `500`: unexpected server failure; `502`: invalid upstream response; `503`: temporary service unavailability; `504`: upstream timeout.
- Prefer accurate status codes and structured error bodies. Do not respond `200` with an `error` string merely to keep success dashboards green.
- Use `Retry-After` for applicable rate limits or temporary unavailability; treat redirects and unsafe methods carefully.
- Track rates by endpoint/status class; do not assume every `4xx` indicates a server outage.
