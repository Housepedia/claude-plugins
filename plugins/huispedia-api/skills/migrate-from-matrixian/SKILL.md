---
name: migrate-from-matrixian
description: "Use when migrating code from the Matrixian API to the Huispedia Data Intelligence API (huispedia.dev): replacing Matrixian calls such as GET /homevalue, GET /homevalue/value, GET /homevalue with your own characteristics, GET /homevalue?nReferences=, GET /leadtool, GET /mx-reference-finder, GET /mx-transaction-finder, GET /woz-value, GET /woz-value/live, GET /mx-house-details, GET /bag-data, GET /buurtinfo. Maps endpoints, authentication, address input, fields and error codes, and lists what has no equivalent."
---

# Migrating from Matrixian

How to move an integration from Matrixian to the Huispedia Data Intelligence API. The mapping below is the one on https://huispedia.dev/en/docs/migrate/matrixian/. For anything beyond the mapping (keys, locators, credits, all endpoints) use the `huispedia-api` skill.

## How to go about it

1. Find every Matrixian call in the code: base URL, endpoints, the key's header and environment variable.
2. Per call, find the Huispedia endpoint in *Endpoints* below. A row without one, or a line under *Not available here*, is a gap: tell the user instead of inventing a replacement.
3. Change authentication to `X-API-Key: hp_…` from a new environment variable (`HUISPEDIA_KEY`); the base URL is `https://huispedia.dev/v5`.
4. Turn the address input into a locator in the path (*Addresses and areas*), and map every field the code reads (*Fields*). Numbers are numbers, not strings.
5. Map error handling by `code` (*Errors*); handle `409 address_ambiguous` by letting the user pick a candidate.
6. Test against the live API with the free monthly credits; there is no sandbox. Contract endpoints answer `403 entitlement_required` until the account has a contract (sales@huispedia.nl).

Based on Matrixian's public documentation, October 2026.

## Endpoints

| Matrixian | What | Huispedia |
|---|---|---|
| `GET /homevalue`, `GET /homevalue/value` | Home valuation | [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation) |
| `GET /homevalue` with your own characteristics | Valuation with your own inputs | [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios) |
| `GET /homevalue?nReferences=` | Valuation with references | [`GET /v5/properties/{locator}/valuation/plus`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation/plus) (contract)<br>With the five best reference homes. |
| `GET /leadtool` | Value range | [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation)<br>The range is `lower` to `upper`. |
| `GET /mx-reference-finder` | Reference homes | [`GET /v5/properties/{locator}/references`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/references) (contract) |
| `GET /mx-transaction-finder` | Sale price | [`GET /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/transactions) (contract) |
| `GET /woz-value`, `GET /woz-value/live` | WOZ value | [`GET /v5/properties/{locator}/woz`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/woz) |
| `GET /mx-house-details` | Property characteristics | [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})<br>[`GET /v5/properties/{locator}/energy-label`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/energy-label) |
| `GET /bag-data` | BAG data | [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})<br>[`GET /v5/buildings/{bag_pand_id}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/buildings/{bag_pand_id}) |
| `GET /buurtinfo` | Neighbourhood information | [`GET /v5/areas/{area}/demographics`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/demographics)<br>[`GET /v5/areas/{area}/stats`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/stats)<br>[`GET /v5/areas/{area}`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}) |

Transactions, reference homes and home valuation plus come with a contract: [sales@huispedia.nl](mailto:sales@huispedia.nl).

## Authentication

|  | Matrixian | Huispedia |
|---|---|---|
| **Key** | `Authorization: Bearer <api-key>` | `Authorization: Bearer hp_…` or `X-API-Key: hp_…` |
| **Base URL** | `https://api.matrixian.com/v1` | `https://huispedia.dev/v5` |
| **Method** | `GET` with query parameters | `GET` with the home in the path; `POST` only for scenarios |
| **Response** | `{id, params, data, error, meta, status}` | The data at the top level; `meta` with `request_id` and `credits` |

## Addresses and areas

| Matrixian | Huispedia | Note |
|---|---|---|
| `postalCode` + `houseNumber` + `houseLetter` + `houseNumberExt` | `1076LJ-12-2` | Postcode, number and extension with hyphens in the path; letter and extension go together in the last part. |
| `houseLetter`, `houseNumberExt` | `house_number_ext` | Not sure: `GET /v5/addresses/1076LJ-12?all=true` returns every home with its `house_number_ext`. |
| `bagId`, `bagNumberId` (nummeraanduiding) | `bag:0363200000028422` |  |
| `bagObjectId`, `bagBuildingId` | `bag_verblijfsobject_id`, `bag_pand_id` | In `GET /v5/addresses/{locator}`; ask for a building with `GET /v5/buildings/{bag_pand_id}`. |
| `valuationDate` (`2026-09-01`) | `as_of_date` (`2026-09-01`) |  |
| `buurtCode` (`BU04170328`) | `neighbourhood:BU04170328` | An area is `level:code`; `GET /v5/areas/lookup?postcode=1076LJ` gives the codes. |

More in [Addresses and areas](https://huispedia.dev/en/docs/locators/).

## Fields

### Home valuation

`GET /homevalue` → [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation)

| Matrixian | Huispedia | Note |
|---|---|---|
| `postalCode`, `houseNumber`, `houseLetter`, `houseNumberExt` | `{locator}` |  |
| `valuationDate` | `?as_of_date=` | Up to six months back; further back with the valuation history. |
| `data.predictedValue`, `data.weightedValue` | `valuation.value` |  |
| `data.valueDate` | `valuation.as_of_date` |  |
| `data.accuracy` | `confidence.score_pct` | Here the probability that the value is within 10% of a sale price. |
| `data.confidenceClass` | `confidence.class` | There 0 to 5; here `high`, `medium` or `low`. |
| `data.weightedValueRange` (Leadtool) | `valuation.lower`, `valuation.upper`, `valuation.interval` | Two numbers with their certainty (80%). |
| `data.bagId`, `postalCode`, `houseNumber`, `houseNumberExt`, `street`, `city`, `municipality` | `address.bag_id`, `address.postcode`, `address.house_number`, `address.house_number_ext`, `address.street`, `address.city`, `address.municipality` |  |
| `data.buildType`, `buildYear`, `useSurface`, `parcelSurface`, `volume`, `energyLabel`, `rooms` | `type.house_type`, `construction.construction_year`, `dimensions.living_area_m2`, `dimensions.plot_area_m2`, `dimensions.volume_m3`, `energy.energy_label`, `rooms.rooms`<br>In [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}) | From Property characteristics. |
| `data.indexedPrice`, `histTransactionPrice`, `histTransactionDate` | `data[].price_indexed`, `data[].price`, `data[].transaction_date`<br>In [`GET /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/transactions) | Every sale of the home, with a contract. |
| `data.references[]` | `references[]`<br>In [`GET /v5/properties/{locator}/valuation/plus`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation/plus) |  |
| `data.numZip*`, `medZip*`, `numStr*`, `medStr*` | `market.transactions_count`, `market.avg_transaction_price`<br>In [`GET /v5/areas/{area}/stats`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/stats) | Per neighbourhood, district or municipality; not per postcode or street. |

### Valuation with your own characteristics

`GET /homevalue` → [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios)

| Matrixian | Huispedia | Note |
|---|---|---|
| `postalCode`, `houseNumber`, `houseNumberExt` | `data[].postcode`, `data[].house_number`, `data[].house_number_ext` |  |
| `useSurface` | `data[].living_area_m2` |  |
| `parcelSurface`, `volume` | `data[].plot_area_m2`, `data[].volume_m3` |  |
| `buildYear`, `numberOfRooms` | `data[].construction_year`, `data[].rooms` |  |
| `buildType` | `data[].house_type` | `terraced`, `corner`, `semi_detached`, `detached` or `apartment`. |
| `energyLabel` | `data[].energy_label` |  |
| `monument` | *Not available here* |  |
| `data.predictedValue` | `data[].valuation.value` |  |

### Property characteristics

`GET /mx-house-details` → [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})

| Matrixian | Huispedia | Note |
|---|---|---|
| `objectType` | `type.object_type` |  |
| `buildYear` | `construction.construction_year` |  |
| `livingSurface`, `useSurface` | `dimensions.living_area_m2`, `dimensions.bag_area_m2` |  |
| `otherIndoorSpace`, `attachedOutdoorSpace`, `externalStorageSpace` | `dimensions.other_indoor_m2`, `dimensions.building_bound_outdoor_m2`, `dimensions.external_storage_m2` |  |
| `parcelSurface`, `volume` | `dimensions.plot_area_m2`, `dimensions.volume_m3` |  |
| `numberOfRooms`, `numberOfBedrooms`, `numberOfBathrooms`, `numberOfToilets` | `rooms.rooms`, `rooms.bedrooms`, `rooms.bathrooms`, `rooms.separate_toilets` |  |
| `energyLabel` | `energy.energy_label` |  |
| `energyLabelRegistrationValidUntil` | `valid_to`<br>In [`GET /v5/properties/{locator}/energy-label`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/energy-label) |  |
| `latitude`, `longitude` | `address.location.lat`, `address.location.lon` |  |
| `bagObjectId`, `bagBuildingId`, neighbourhood and municipality codes | `bag_verblijfsobject_id`, `bag_pand_id`, `neighbourhood.code`, `municipality_code`<br>In [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator}) |  |
| `description` | *Not available here* |  |

### Reference homes

`GET /mx-reference-finder` → [`GET /v5/properties/{locator}/references`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/references)

| Matrixian | Huispedia | Note |
|---|---|---|
| `nReferences` | `?limit=` | 3 to 30. |
| `valuationDate` | `?as_of_date=` |  |
| `maxDistance`, `minTransportDate` | `?max_distance_m=`, `?max_age_months=` |  |
| `minUseSurface`, `maxUseSurface` | `?min_living_area_m2=`, `?max_living_area_m2=` |  |
| `minBuildYear`, `maxBuildYear` | `?min_construction_year=`, `?max_construction_year=` |  |
| `allowedHouseTypes` | `?same_type=` | Only the same type, or all types. |
| `locationWeight`, `houseDetailsWeight`, `transportDateWeight` | `?sort=` | Sort by `similarity`, `distance` or `date`; no weights of your own. |
| `references[]` | `data[]` |  |
| `distance`, `dateSale` | `data[].distance_m`, `data[].transaction_date` |  |
| `houseType`, `buildYear`, `useSurface` | `data[].house_type`, `data[].construction_year`, `data[].living_area_m2` |  |
| `referenceScore`, `locationScore`, `houseDetailsScore` | *Not available here* |  |
| `offeredPrice`, `imageFront`, `description` | *Not available here* |  |

### Transactions

`GET /mx-transaction-finder` → [`GET /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/transactions)

| Matrixian | Huispedia | Note |
|---|---|---|
| `valuationDate`, `dateTransport` | *Not available here* | You get every sale of the home; filter on `transaction_date` yourself. |
| `saleDate`, `transportDate` | `data[].price_agreement_date`, `data[].transaction_date` |  |
| `transactionPrice` | `data[].price` |  |
| `transactionPriceIndexed` | `data[].price_indexed` | At today's price level. |

### WOZ value

`GET /woz-value` → [`GET /v5/properties/{locator}/woz`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/woz)

| Matrixian | Huispedia | Note |
|---|---|---|
| `wozValues[].valuationDate`, `wozValue` | `values[].reference_date`, `values[].value` |  |
| `characteristics` | `type`, `dimensions`, `construction`<br>In [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}) |  |

### Neighbourhood information

`GET /buurtinfo` → [`GET /v5/areas/{area}/demographics`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/demographics)

| Matrixian | Huispedia | Note |
|---|---|---|
| `buurtCode` or an address | `{area}` | Codes from `GET /v5/areas/lookup?postcode=`. |
| `Bevolking` | `population`, `households`, `age` |  |
| `Wonen` | `stock`<br>In [`GET /v5/areas/{area}/stats`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/stats) |  |
| `geometry` | `geometry`<br>In [`GET /v5/areas/{area}`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}) | With `include=geometry`. |
| `Inkomen`, `Energie`, `Criminaliteit` | *Not available here* |  |

## Errors

An error is `application/problem+json` with a fixed `code`; every code is on [Errors](https://huispedia.dev/en/docs/errors/).

| Matrixian | Huispedia | Note |
|---|---|---|
| `400` (`errors[].reason` is the parameter) | [`400 invalid_request`](https://huispedia.dev/en/docs/errors/#invalid_request) | `errors` names what is wrong per field. |
| `401` | [`401 unauthenticated`](https://huispedia.dev/en/docs/errors/#unauthenticated), [`401 key_revoked`](https://huispedia.dev/en/docs/errors/#key_revoked), [`401 key_expired`](https://huispedia.dev/en/docs/errors/#key_expired) |  |
| `403` (`access_denied`, `missing_subscription`) | [`403 entitlement_required`](https://huispedia.dev/en/docs/errors/#entitlement_required), [`403 forbidden`](https://huispedia.dev/en/docs/errors/#forbidden) |  |
| `403` (`rate_limit`) | [`429 rate_limited`](https://huispedia.dev/en/docs/errors/#rate_limited) | Here `429`; wait `Retry-After` seconds. |
| `404` (unknown address) | [`404 not_found`](https://huispedia.dev/en/docs/errors/#not_found) |  |
| `500` | [`500 internal_error`](https://huispedia.dev/en/docs/errors/#internal_error) |  |
| `503` | [`503 upstream_unavailable`](https://huispedia.dev/en/docs/errors/#upstream_unavailable) |  |
| `{id, status: "failure", error: {code, message, errors}}` | `application/problem+json` with `code`, `detail`, `errors`, `request_id` |  |

## Bulk and limits

|  | Matrixian | Huispedia |
|---|---|---|
| **Rate** | Not published; at the limit `403 rate_limit` | [6 per second and 6 at once on a paid plan; 1 and 2 on Free](https://huispedia.dev/en/docs/rate-limits/); `X-RateLimit-Remaining` in every response |
| **Bulk** | No bulk endpoint documented | `POST /v5/valuations/scenarios`: 100 homes per call |
| **Correlation** | `id`, `X-Amzn-Trace-Id` | `meta.request_id`, `request_id` |

## Example: home valuation

**Matrixian**

```bash
curl "https://api.matrixian.com/v1/homevalue?postalCode=1076LJ&houseNumber=12&houseNumberExt=2" \
  -H "Authorization: Bearer $MATRIXIAN_KEY"
```

```python
import os, requests

r = requests.get(
    "https://api.matrixian.com/v1/homevalue",
    headers={"Authorization": f"Bearer {os.environ['MATRIXIAN_KEY']}"},
    params={"postalCode": "1076LJ", "houseNumber": 12, "houseNumberExt": "2"},
)
value = r.json()["data"]["predictedValue"]
```

**Huispedia**

```bash
curl https://huispedia.dev/v5/properties/1076LJ-12-2/valuation \
  -H "Authorization: Bearer $HUISPEDIA_KEY"
```

```python
import os, requests

r = requests.get(
    "https://huispedia.dev/v5/properties/1076LJ-12-2/valuation",
    headers={"Authorization": f"Bearer {os.environ['HUISPEDIA_KEY']}"},
)
r.raise_for_status()
v = r.json()["valuation"]
value, lower, upper = v["value"], v["lower"], v["upper"]
```

## Differences

### Not available here

- Checking availability first (`/homevalue/available`): not available here; a `404 not_found` costs no credits.
- Monument status as valuation input (`monument`): not available here.
- Market figures per postcode or street (`numZip*`, `numStr*`): not available here; per neighbourhood instead.
- Your own weights and scores per reference home: not available here; the order gives the likeness.
- Income, energy and crime per neighbourhood: not available here.
- Housing costs (`/local-tax-energy-expenses`): not available here.
- Home photos (`/house-images`, `/get_front_view`): not available here.
- Cadastral objects and ownership extracts: not available here.
- Commercial transactions: not available here.

### No equivalent at Matrixian

- An interval with lower and upper bound on every valuation: no equivalent at Matrixian, except the Leadtool's range string.
- `409 address_ambiguous` with the candidates when an address has several homes: no equivalent at Matrixian.
- What-if scenarios and up to 100 homes per call: no equivalent at Matrixian.
- Monthly valuation history: no equivalent at Matrixian.
- Price index per area (`/v5/areas/{area}/trends`): no equivalent in Matrixian's public documentation.
- Environment (foundation, subsidence, noise, air) per address: no equivalent in Matrixian's public documentation.

## Sources

- [Matrixian API, OpenAPI](https://matrixian-api.s3.eu-west-3.amazonaws.com/openapi.yaml)
- [API-authenticatie](https://docs.matrixian.com/readme/api-authenticatie)
- [API-responsestructuur](https://docs.matrixian.com/readme/api-response-structuur)
- [API-foutmeldingen](https://docs.matrixian.com/readme/api-foutmeldingen)
- [Woningwaarde API](https://docs.matrixian.com/waarderen-en-taxeren/woningwaarde-api-1)
- [Woningwaarde API (alleen waarde)](https://docs.matrixian.com/waarderen-en-taxeren/woningwaarde-api)
- [Leadtool API](https://docs.matrixian.com/waarderen-en-taxeren/leadtool-api)
- [Referentiewoningen API](https://docs.matrixian.com/waarderen-en-taxeren/referentiewoningen-api)
- [Koopsom API](https://docs.matrixian.com/waarderen-en-taxeren/koopsom-api)
- [WOZ-waarde API](https://docs.matrixian.com/waarderen-en-taxeren/woz-waarde-api)
- [Woningkenmerken API](https://docs.matrixian.com/woningdata/woningkenmerken-api)
- [BAG API](https://docs.matrixian.com/woningdata/bag-api)
- [Buurtinfo API](https://docs.matrixian.com/woningdata/buurtinfo-api)
