# Errors

> Every error code and what to do about it.

Source: https://huispedia.dev/en/docs/errors/

An error is `application/problem+json` (RFC 9457) with a fixed `code`; `detail` says in your `Accept-Language` what you can do. An error costs no credits.

```json
{
  "type": "https://huispedia.dev/en/docs/errors#insufficient_credits",
  "title": "Insufficient credits",
  "status": 402,
  "code": "insufficient_credits",
  "detail": "This call costs 8 credits; your balance is 3. Buy credits in the dashboard.",
  "request_id": "req_01M4…",
  "errors": []
}
```

| HTTP | `code` | When |
|---|---|---|
| 400 | `invalid_request` | The request is not right; `errors` says what is wrong per field |
| 401 | `unauthenticated` | Missing or unknown key |
| 401 | `key_revoked`, `key_expired` | Key revoked or expired |
| 402 | `insufficient_credits` | Your balance does not cover the call |
| 402 | `spend_cap_reached` | Your spend cap for this month is reached |
| 403 | `entitlement_required` | This endpoint is not part of your account |
| 403 | `ip_not_allowed` | IP address outside the key's allowlist |
| 403 | `account_suspended` | Account suspended |
| 403 | `forbidden` | This account may not do this |
| 404 | `not_found` | The locator finds nothing |
| 405 | `method_not_allowed` | This method does not exist on this path |
| 409 | `address_ambiguous` | Several homes at the address; `errors` carries the candidates |
| 409 | `conflict` | The action clashes with the current state; `detail` says how |
| 422 | `data_unavailable` | The home or area exists, but this data is not available for it |
| 429 | `rate_limited` | Per-key or per-account limit; see `Retry-After` |
| 503 | `upstream_unavailable` | Temporarily unavailable; see `Retry-After` |
| 503 | `billing_unavailable` | Payments are not possible right now |
| 500 | `internal_error` | A fault on our side; quote the `request_id` |
