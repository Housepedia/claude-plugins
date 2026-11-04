---
name: migrate-from-huispedia-v2
description: "Use when migrating code from the Huispedia v2 API to the Huispedia Data Intelligence API (huispedia.dev): replacing Huispedia v2 calls such as GET /v2/address/lookup/{postcode}/{houseNumber}/{houseNumberExt?}, GET /v2/address/validate/{postcode}/{houseNumber}/{houseNumberExt?}, GET /v2/address/autocomplete/{query}, GET /v2/properties/{postcode}/{houseNumber}/{houseNumberExt?}, GET /v2/properties/{id}, GET /v2/properties?filter[…], GET /v2/homevalue, GET /v2/homevalue with your own characteristics, GET /v2/info. Maps endpoints, authentication, address input, fields and error codes, and lists what has no equivalent."
---

# Migrating from Huispedia v2

How to move an integration from Huispedia v2 to the Huispedia Data Intelligence API. The mapping below is the one on https://huispedia.dev/en/docs/migrate/huispedia-v2/. For anything beyond the mapping (keys, locators, credits, all endpoints) use the `huispedia-api` skill.

## How to go about it

1. Find every Huispedia v2 call in the code: base URL, endpoints, the key's header and environment variable.
2. Per call, find the Huispedia endpoint in *Endpoints* below. A row without one, or a line under *Not available here*, is a gap: tell the user instead of inventing a replacement.
3. Change authentication to `X-API-Key: hp_…` from a new environment variable (`HUISPEDIA_KEY`); the base URL is `https://huispedia.dev/v5`.
4. Turn the address input into a locator in the path (*Addresses and areas*), and map every field the code reads (*Fields*). Numbers are numbers, not strings.
5. Map error handling by `code` (*Errors*); handle `409 address_ambiguous` by letting the user pick a candidate.
6. Test against the live API with the free monthly credits; there is no sandbox. Contract endpoints answer `403 entitlement_required` until the account has a contract (sales@huispedia.nl).

Based on the documentation of the old Huispedia API (v2, `api.huispedia.nl`) and how it behaved, November 2026.

## Endpoints

| Huispedia v2 | What | Huispedia |
|---|---|---|
| `GET /v2/address/lookup/{postcode}/{houseNumber}/{houseNumberExt?}` | Addresses at a postcode and house number | [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})<br>With `?all=true` you get every home at the house number. |
| `GET /v2/address/validate/{postcode}/{houseNumber}/{houseNumberExt?}` | Address validation | [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})<br>One home: the address; several: `409 address_ambiguous` with the candidates; none: `404 not_found`. |
| `GET /v2/address/autocomplete/{query}` | Address autocomplete | [`GET /v5/addresses/suggest`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/suggest)<br>Free; the text goes in `?q=`. |
| `GET /v2/properties/{postcode}/{houseNumber}/{houseNumberExt?}` | Property characteristics | [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})<br>[`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})<br>BAG ids and CBS codes are with the address. |
| `GET /v2/properties/{id}` | Home by Huispedia number | [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})<br>With the BAG id as locator: `bag:0363200000028422`. |
| `GET /v2/properties?filter[…]` | Search homes by characteristics | *Not available here* |
| `GET /v2/homevalue` | Home valuation | [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation)<br>We know the characteristics; you only pass the address. |
| `GET /v2/homevalue` with your own characteristics | Valuation with your own inputs | [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios) |
| `GET /v2/info` | Subscription and usage | [`GET /v5/account`](https://huispedia.dev/en/docs/endpoints/#tag/account/GET/v5/account)<br>[`GET /v5/account/usage`](https://huispedia.dev/en/docs/endpoints/#tag/account/GET/v5/account/usage) |

## Authentication

|  | Huispedia v2 | Huispedia |
|---|---|---|
| **Account** | Key and subscription from Huispedia | An account on huispedia.dev with 200 free credits a month; your old key does not work here |
| **Key** | `X-Api-Key: <key>` | `X-API-Key: hp_…` or `Authorization: Bearer hp_…`; you create it yourself in the [dashboard](https://huispedia.dev/en/dashboard/) |
| **Base URL** | `https://api.huispedia.nl/v2` | `https://huispedia.dev/v5` |
| **Method** | `GET`; the address in the path, in the query for the valuation | `GET` with the home always in the path; `POST` only for scenarios |
| **Response** | `{data: [{type, id, attributes}], links, meta}` | The data at the top level; `meta` with `request_id` and `credits` |

## Addresses and areas

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `/1076LJ/12/2` | `1076LJ-12-2` | Postcode, number and extension with hyphens in one part of the path; case and spaces do not matter. |
| `postcode`, `house_number`, `house_number_ext` (`/homevalue`) | `1076LJ-12-2` |  |
| `kadasterData.bagNumberDesignationId` | `bag:0363200000028422` |  |
| `id` (`182443`) | `bag:0363200000028422` | The Huispedia number does not exist here; keep the BAG id (`bag_id`). |
| `cbsNeighbourhoodId`, `cbsDistrictId` | `neighbourhood:BU0363KH05`, `district:WK0363KH` | An area is `level:code`; `GET /v5/areas/lookup?postcode=1076LJ` gives the codes. |

More in [Addresses and areas](https://huispedia.dev/en/docs/locators/).

## Fields

### Addresses

`GET /v2/address/lookup` → [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `{postcode}/{houseNumber}/{houseNumberExt?}` | `{locator}`, `?all=` | Without extension and with `all=true` you get every home at the house number. |
| `data[].id` | `data[].bag_id` |  |
| `street`, `houseNumber`, `houseNumberExt`, `postcode`, `city`, `municipality`, `province` | `data[].street`, `data[].house_number`, `data[].house_number_ext`, `data[].postcode`, `data[].city`, `data[].municipality`, `data[].province` |  |
| `houseNumberFull` | `data[].house_number`, `data[].house_number_ext` | Join them yourself: `12-2`. |
| `latitude`, `longitude` | `data[].location.lat`, `data[].location.lon` |  |
| `cbsNeighbourhoodId`, `cbsDistrictId` | `data[].neighbourhood.code`, `data[].district.code` | Here the CBS code as text (`BU0363KH05`), there a number. |
| `links.related` | `data[].bag_id` | Ask for the home with `bag:<bag_id>` as locator. |
| `meta.totalResults`, `page[number]`, `page[size]` | *Not available here* | All homes in one answer. |

### Property characteristics

`GET /v2/properties/{postcode}/{houseNumber}` → [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `{postcode}/{houseNumber}/{houseNumberExt?}` | `{locator}` | Does the address have several homes: `409 address_ambiguous` with the candidates. |
| `street`, `houseNumber`, `houseNumberExt`, `postcode`, `city`, `municipality`, `province` | `address.street`, `address.house_number`, `address.house_number_ext`, `address.postcode`, `address.city`, `address.municipality`, `address.province` |  |
| `latitude`, `longitude` | `address.location.lat`, `address.location.lon` |  |
| `useSurface` | `dimensions.living_area_m2` |  |
| `plotSize`, `volume` | `dimensions.plot_area_m2`, `dimensions.volume_m3` |  |
| `buildYear` | `construction.construction_year` |  |
| `houseType`, `houseSubType` | `type.object_type`, `type.house_type`, `type.house_type_detail` | There Dutch text (`Woonhuis`, `Vrijstaande woning`); here `house` or `apartment` and `terraced`, `corner`, `semi_detached`, `detached`, `apartment` or `other`. The Dutch type is in `house_type_detail`. |
| `energyLabel` | `energy.energy_label` | `A+` instead of `APLUS`. |
| `huispediaProfileUrl` | `address.huispedia_url` |  |
| `kadasterData.bagNumberDesignationId` | `address.bag_id` |  |
| `kadasterData.bagBuildingId`, `bagObjectId`, `cbsNeighbourhoodId`, `cbsDistrictId` | `bag_pand_id`, `bag_verblijfsobject_id`, `neighbourhood.code`, `district.code`<br>In [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator}) |  |
| `predictedValue`, `predictedValueRange` | `valuation.value`, `valuation.lower`, `valuation.upper`<br>In [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation) |  |
| `offer.status`, `salePrice`, `rentalPrice` | *Not available here* |  |
| `image.href` | *Not available here* |  |

### Home valuation

`GET /v2/homevalue` → [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation)

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `postcode`, `house_number`, `house_number_ext` | `{locator}` |  |
| `use_surface` | *Not available here* | No longer needed; to use your own area, make it a scenario. |
| `data.predictedValue` | `valuation.value` |  |
| `data.predictedValueRange.lowerBound`, `upperBound` | `valuation.lower`, `valuation.upper`, `valuation.interval` | There at most 10% from the value; here an interval with 80% certainty. |
| Address and property fields in `data` | `address` | Characteristics from Property characteristics. |

### Valuation with your own characteristics

`GET /v2/homevalue` → [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios)

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `postcode`, `house_number`, `house_number_ext` | `data[].postcode`, `data[].house_number`, `data[].house_number_ext` | Or `locator`: `1076LJ-12-2`. |
| `use_surface` | `data[].living_area_m2` | Optional; without it the known area applies. |
| `plot_size`, `volume` | `data[].plot_area_m2`, `data[].volume_m3` |  |
| `build_year` | `data[].construction_year` |  |
| `type` | `data[].house_type` | `appartement` → `apartment`, `tussenwoning` → `terraced`, `hoekwoning` → `corner`, `2-onder-1-kapwoning` → `semi_detached`, `vrijstaande woning` → `detached`; leave out `other`. |
| `energy_label` | `data[].energy_label` | `A+` to `A++++` instead of `APLUS…`; leave out `UNKNOWN` and `NOT_REQUIRED`. |
| `maintenance` | `data[].maintenance_overall` | 1 to 5: `POOR` 1, `BELOW AVERAGE` 2, `AVERAGE` 3, `GOOD` 4, `EXCELLENT` 5. |
| `data.predictedValue` | `data[].valuation.value` |  |
| `data.predictedValueRange.lowerBound`, `upperBound` | `data[].valuation.lower`, `data[].valuation.upper` |  |

## Errors

An error is `application/problem+json` with a fixed `code`; every code is on [Errors](https://huispedia.dev/en/docs/errors/).

| Huispedia v2 | Huispedia | Note |
|---|---|---|
| `400` (`invalid postcode format`, `invalid number format`), `422` (`/homevalue`) | [`400 invalid_request`](https://huispedia.dev/en/docs/errors/#invalid_request) | Always `400`; `errors` names what is wrong per field. |
| `401` (`Unauthenticated.`) | [`401 unauthenticated`](https://huispedia.dev/en/docs/errors/#unauthenticated), [`401 key_revoked`](https://huispedia.dev/en/docs/errors/#key_revoked), [`401 key_expired`](https://huispedia.dev/en/docs/errors/#key_expired), [`403 ip_not_allowed`](https://huispedia.dev/en/docs/errors/#ip_not_allowed) |  |
| `403` (`Access forbidden, please check your current subscriptions`) | [`403 entitlement_required`](https://huispedia.dev/en/docs/errors/#entitlement_required) |  |
| `403` (`You run out of credits`) | [`402 insufficient_credits`](https://huispedia.dev/en/docs/errors/#insufficient_credits), [`402 spend_cap_reached`](https://huispedia.dev/en/docs/errors/#spend_cap_reached) | Here `402`. |
| `404` (`address not found`, `no addresses found`, `the address does not exist`, `Address is not valid`) | [`404 not_found`](https://huispedia.dev/en/docs/errors/#not_found) |  |
| None; `validate` and `/properties` took the first home | [`409 address_ambiguous`](https://huispedia.dev/en/docs/errors/#address_ambiguous) | The candidates are in `errors`; pick one and ask again. |
| `429` (`Too Many Attempts.`) | [`429 rate_limited`](https://huispedia.dev/en/docs/errors/#rate_limited) | Wait `Retry-After` seconds. |
| `500` | [`500 internal_error`](https://huispedia.dev/en/docs/errors/#internal_error), [`503 upstream_unavailable`](https://huispedia.dev/en/docs/errors/#upstream_unavailable) |  |
| `{"error": "…"}`, `{"message": "…"}` | `application/problem+json` with `code`, `detail`, `errors`, `request_id` |  |

## Bulk and limits

|  | Huispedia v2 | Huispedia |
|---|---|---|
| **Rate** | 120 per minute | [6 per second and 6 at once on a paid plan; 1 and 2 on Free](https://huispedia.dev/en/docs/rate-limits/); `X-RateLimit-Remaining` in every response |
| **Usage** | Calls per month per subscription, errors included; `X-Subscription-RateLimit-Remaining` | [Credits per endpoint](https://huispedia.dev/en/docs/credits/); only a 2xx answer costs credits; `meta.credits` with what the call cost and your balance |
| **Bulk** | One home per call | `POST /v5/valuations/scenarios`: 100 homes per call |
| **Correlation** | None | `meta.request_id`, `request_id` |

## Example: home valuation

**Huispedia v2**

```bash
curl "https://api.huispedia.nl/v2/homevalue?postcode=1076LJ&house_number=12&house_number_ext=2&use_surface=85" \
  -H "X-Api-Key: $HUISPEDIA_V2_KEY"
```

```python
import os, requests

r = requests.get(
    "https://api.huispedia.nl/v2/homevalue",
    headers={"X-Api-Key": os.environ["HUISPEDIA_V2_KEY"]},
    params={"postcode": "1076LJ", "house_number": 12, "house_number_ext": "2", "use_surface": 85},
)
d = r.json()["data"]
value = d["predictedValue"]
lower, upper = d["predictedValueRange"]["lowerBound"], d["predictedValueRange"]["upperBound"]
```

**Huispedia**

```bash
curl https://huispedia.dev/v5/properties/1076LJ-12-2/valuation \
  -H "X-API-Key: $HUISPEDIA_KEY"
```

```python
import os, requests

r = requests.get(
    "https://huispedia.dev/v5/properties/1076LJ-12-2/valuation",
    headers={"X-API-Key": os.environ["HUISPEDIA_KEY"]},
)
r.raise_for_status()
v = r.json()["valuation"]
value, lower, upper = v["value"], v["lower"], v["upper"]
```

## Differences

### Not available here

- Searching homes by characteristics (`GET /v2/properties?filter[…]`): not available here.
- The Huispedia number of a home (`id`): not available here; use the BAG id.
- Listings with status, asking and rental price (`offer`): not available here.
- Home photo (`image`): not available here.
- House type `other` as valuation input: not available here; leave out `house_type` and the known value applies.

### New in v5

- How certain a value is (`confidence`) and an interval with 80% certainty: no equivalent in v2.
- `409 address_ambiguous` with the candidates when an address has several homes: no equivalent in v2.
- WOZ value, energy label with validity, transactions, reference homes and valuation history: no equivalent in v2.
- What-if scenarios and up to 100 homes per call: no equivalent in v2.
- Neighbourhood statistics, price index, demographics and environment: no equivalent in v2.

## Sources

- [Huispedia API V2 documentation (PDF)](https://huispedia.nl/files/Huispedia-API-documentation.pdf)
