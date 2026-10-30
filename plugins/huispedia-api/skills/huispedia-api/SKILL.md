---
name: huispedia-api
description: Use when writing code that calls the Huispedia Data Intelligence API (huispedia.dev, /v5) or when someone needs Dutch property data by address — home valuations (AVM), WOZ value, energy label, property characteristics, transactions, reference homes, neighbourhood statistics, price index, demographics. Covers hp_ API keys, address and area locators, the endpoints and what each costs in credits, rate limits and error codes.
---

# Huispedia Data Intelligence API

Address-first property data for the Netherlands over REST. Base URL `https://huispedia.dev/v5`; JSON in and out.

## Authentication

Send the key as `X-API-Key: hp_…` or `Authorization: Bearer hp_…`, never in the URL. Read it from the environment (`HUISPEDIA_KEY` in the examples); never hard-code or log it. A key is created once in the dashboard (https://huispedia.dev/en/dashboard/) and shown once. More: [authentication](references/authentication.md).

## Pointing at a home or an area

- A home is `{locator}` in the path: `bag:0363200000028422` (BAG nummeraanduiding) or `1076LJ-12-2` (postcode, house number, extension).
- Unsure of the extension: `GET /v5/addresses/suggest?q=…` (free) or `GET /v5/addresses/1076LJ-12?all=true`. A `409 address_ambiguous` carries the candidates in `errors`; pick one and retry with its locator.
- An area is `{area}` as `level:code`: `neighbourhood:BU0363KH05`, `municipality:GM0363`, `postcode4:1076`, `national`. `GET /v5/areas/lookup?postcode=1076LJ` gives the codes.

More: [addresses and areas](references/locators.md).

## Endpoints

Every endpoint with its credits, parameters and top-level response fields: [endpoints](references/endpoints.md). The full schema is the OpenAPI document at https://huispedia.dev/openapi.json; the live catalogue with prices at https://huispedia.dev/v5/catalog (no key needed).

## First call

```bash
curl https://huispedia.dev/v5/properties/1076LJ-12-2/valuation -H "X-API-Key: $HUISPEDIA_KEY"
```

`valuation.value` is the home value; `valuation.lower` and `valuation.upper` the 80% interval; `meta.credits` what the call cost and the balance left. More: [quickstart](references/quickstart.md).

## Credits

Each endpoint costs a fixed number of credits. Only a 2xx answer is charged; errors and `HEAD` are free. Some endpoints (transactions, reference homes, valuation plus) need a contract; without one they answer `403 entitlement_required`. More: [credits](references/credits.md), [payments](references/billing.md).

## Errors and limits

Errors are `application/problem+json` with a fixed `code`, a `detail` to show the user and a `request_id`. Branch on `code`, not on the text:

- `429 rate_limited` and `503 upstream_unavailable`: wait `Retry-After` seconds, then retry.
- `402 insufficient_credits` / `spend_cap_reached`: stop; the account needs credits or a higher cap.
- `404 not_found`: the locator finds nothing; `422 data_unavailable`: the home exists but this data does not.

Limits are per account and plan; every answer carries `X-RateLimit-Remaining` and `X-RateLimit-Reset`. Respect `Cache-Control`. More: [errors](references/errors.md), [rate limits](references/rate-limits.md).

Within v5 the API only grows: ignore fields and enum values you do not know ([versioning](references/versioning.md), [changelog](references/changelog.md)).

Migrating from Altum or Matrixian: use the `migrate-from-altum` or `migrate-from-matrixian` skill.
