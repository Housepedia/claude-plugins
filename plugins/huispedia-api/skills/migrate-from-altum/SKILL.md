---
name: migrate-from-altum
description: "Use when migrating code from the Altum API to the Huispedia Data Intelligence API (huispedia.dev): replacing Altum calls such as POST /avm, POST /v2/avm, POST /avm with your own characteristics, POST /avm/bulk, POST /avmplus, POST /v2/objectdata, POST /interactive-reference, POST /v2/transaction, POST /v2/transaction/ping, POST /woz, GET /woz, GET /market-trend, POST /location-data, GET /autosuggest/v2, GET /autosearch, POST /condition-score/{img,url}, GET /credit. Maps endpoints, authentication, address input, fields and error codes, and lists what has no equivalent."
---

# Migrating from Altum

How to move an integration from Altum to the Huispedia Data Intelligence API. The mapping below is the one on https://huispedia.dev/en/docs/migrate/altum/. For anything beyond the mapping (keys, locators, credits, all endpoints) use the `huispedia-api` skill.

## How to go about it

1. Find every Altum call in the code: base URL, endpoints, the key's header and environment variable.
2. Per call, find the Huispedia endpoint in *Endpoints* below. A row without one, or a line under *Not available here*, is a gap: tell the user instead of inventing a replacement.
3. Change authentication to `X-API-Key: hp_…` from a new environment variable (`HUISPEDIA_KEY`); the base URL is `https://huispedia.dev/v5`.
4. Turn the address input into a locator in the path (*Addresses and areas*), and map every field the code reads (*Fields*). Numbers are numbers, not strings.
5. Map error handling by `code` (*Errors*); handle `409 address_ambiguous` by letting the user pick a candidate.
6. Test against the live API with the free monthly credits; there is no sandbox. Contract endpoints answer `403 entitlement_required` until the account has a contract (sales@huispedia.nl).

Based on Altum's public documentation, October 2026.

## Endpoints

| Altum | What | Huispedia |
|---|---|---|
| `POST /avm`, `POST /v2/avm` | Home valuation | [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation) |
| `POST /avm` with your own characteristics | Valuation with your own inputs | [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios) |
| `POST /avm/bulk` | Bulk valuation | [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios)<br>Up to 100 homes per call, answered at once. |
| `POST /avmplus` | Home valuation plus | [`GET /v5/properties/{locator}/valuation/plus`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation/plus) (contract) |
| `POST /v2/objectdata` | Property characteristics | [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}) |
| `POST /interactive-reference` | Reference homes | [`GET /v5/properties/{locator}/references`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/references) (contract) |
| `POST /v2/transaction` | Transactions | [`GET /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/transactions) (contract) |
| `POST /v2/transaction/ping` | Any transactions? | [`HEAD /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/HEAD/v5/properties/{locator}/transactions) |
| `POST /woz`, `GET /woz` | WOZ value | [`GET /v5/properties/{locator}/woz`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/woz) |
| `GET /market-trend` | Market trend | [`GET /v5/areas/{area}/stats`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/stats)<br>[`GET /v5/areas/{area}/trends`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/trends) |
| `POST /location-data` | Location data | [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator})<br>[`GET /v5/areas/{area}/demographics`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/demographics) |
| `GET /autosuggest/v2`, `GET /autosearch` | Address search | [`GET /v5/addresses/suggest`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/suggest) |
| `POST /condition-score/{img,url}` | Condition score | [`GET /v5/properties/{locator}/condition`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/condition)<br>Per address; you do not send your own photos. |
| `GET /credit` | Usage | [`GET /v5/account/usage`](https://huispedia.dev/en/docs/endpoints/#tag/account/GET/v5/account/usage) |

Transactions, reference homes and home valuation plus come with a contract: [sales@huispedia.nl](mailto:sales@huispedia.nl).

## Authentication

|  | Altum | Huispedia |
|---|---|---|
| **Key** | `x-api-key: <key>` | `X-API-Key: hp_…` or `Authorization: Bearer hp_…` |
| **Base URL** | `https://api.altum.ai` | `https://huispedia.dev/v5` |
| **Method** | `POST` with a JSON body | `GET` with the home in the path; `POST` only for scenarios |
| **Testing** | `/sandbox/` in the path | No sandbox: you test with your free credits or on the [Endpoints](https://huispedia.dev/en/docs/endpoints/) page |

## Addresses and areas

| Altum | Huispedia | Note |
|---|---|---|
| `postcode` + `housenumber` + `houseaddition` | `1076LJ-12-2` | Postcode, number and extension with hyphens in the path. |
| `houseaddition` (`A02`, `2B`) | `-A02`, `-2B` | Not sure: `GET /v5/addresses/1076LJ-12?all=true` returns every home with its `house_number_ext`. |
| `bag_id` (verblijfsobject) | `bag:0363200000028422` (nummeraanduiding) | The verblijfsobject id is `bag_verblijfsobject_id` in `GET /v5/addresses/{locator}`. |
| `valuationdate` (`20260901`) | `as_of_date` (`2026-09-01`) |  |
| `postcode` (Markttrend) | `neighbourhood:BU0363KH05` | An area is `level:code`; `GET /v5/areas/lookup?postcode=1076LJ` gives the codes. |

More in [Addresses and areas](https://huispedia.dev/en/docs/locators/).

## Fields

### Home valuation

`POST /avm` → [`GET /v5/properties/{locator}/valuation`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/valuation)

| Altum | Huispedia | Note |
|---|---|---|
| `postcode`, `housenumber`, `houseaddition` | `{locator}` |  |
| `valuationdate` | `?as_of_date=` | Up to six months back; further back with the valuation history. |
| `Output.PriceEstimation` | `valuation.value` | A number, not a string. |
| `Output.Confidence` | `valuation.lower`, `valuation.upper`, `valuation.interval` | There a 90% interval in one string; here an 80% interval in two numbers. |
| `Output.AccuracyIndicator` | `confidence.class`, `confidence.score_pct`, `confidence.within_10_pct` | There a class 0 to 7; here `high`, `medium` or `low` and percentages. |
| `Output.ValuationDate` | `valuation.as_of_date` |  |
| `Output.BagID`, `PostCode`, `HouseNumber`, `HouseAddition`, `Street`, `City` | `address.bag_id`, `address.postcode`, `address.house_number`, `address.house_number_ext`, `address.street`, `address.city` |  |
| `Output.Longitude`, `Latitude` | `address.location.lon`, `address.location.lat` |  |
| `Output.HouseType`, `BuildYear`, `InnerSurfaceArea`, `OuterSurfaceArea`, `Volume`, `EnergyLabel` | `type.house_type`, `construction.construction_year`, `dimensions.living_area_m2`, `dimensions.plot_area_m2`, `dimensions.volume_m3`, `energy.energy_label`<br>In [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}) | From Property characteristics. |
| `Output.Image` | *Not available here* |  |

### Valuation with your own characteristics

`POST /avm` → [`POST /v5/valuations/scenarios`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/POST/v5/valuations/scenarios)

| Altum | Huispedia | Note |
|---|---|---|
| `postcode`, `housenumber`, `houseaddition` | `data[].postcode`, `data[].house_number`, `data[].house_number_ext` |  |
| `innersurfacearea` | `data[].living_area_m2` |  |
| `outersurfacearea` | `data[].plot_area_m2` |  |
| `buildyear` | `data[].construction_year` |  |
| `housetype` | `data[].house_type` | `terraced`, `corner`, `semi_detached`, `detached` or `apartment`. |
| `energylabel` | `data[].energy_label` |  |
| `Output.PriceEstimation` | `data[].valuation.value` |  |

### Property characteristics

`POST /v2/objectdata` → [`GET /v5/properties/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator})

| Altum | Huispedia | Note |
|---|---|---|
| `house_type_generic`, `house_type_specific` | `type.object_type`, `type.house_type_detail` |  |
| `inner_surface_area`, `outer_surface_area`, `volume` | `dimensions.living_area_m2`, `dimensions.plot_area_m2`, `dimensions.volume_m3` |  |
| `build_year` | `construction.construction_year` |  |
| `rooms`, `number_of_bathrooms` | `rooms.rooms`, `rooms.bathrooms` |  |
| `garden_orientation`, `garden_size` | `outdoor.garden_orientation`, `outdoor.garden_area_m2` |  |
| `monument` | `construction.is_monument` |  |
| `longitude`, `latitude`, `municipality_name`, `province` | `address.location.lon`, `address.location.lat`, `address.municipality`, `address.province` |  |
| `neighbourhood_code`, `municipality_code` | `neighbourhood.code`, `municipality_code`<br>In [`GET /v5/addresses/{locator}`](https://huispedia.dev/en/docs/endpoints/#tag/addresses/GET/v5/addresses/{locator}) |  |
| `ozb_tax`, `waste_tax_*`, `sewage_tax_*` | *Not available here* |  |

### Reference homes

`POST /interactive-reference` → [`GET /v5/properties/{locator}/references`](https://huispedia.dev/en/docs/endpoints/#tag/valuations/GET/v5/properties/{locator}/references)

| Altum | Huispedia | Note |
|---|---|---|
| `reference_number` | `?limit=` | 3 to 30. |
| `date_limit` | `?max_age_months=` |  |
| `housetype_to_search`, `strict_*`, `comparable_*` | `?same_type=`, `?max_distance_m=`, `?min_living_area_m2=`, `?max_living_area_m2=`, `?min_construction_year=`, `?max_construction_year=` | Bounds instead of levels. |
| `weight_*` | `?sort=` | Sort by `similarity`, `distance` or `date`; no weights of your own. |
| `GivenHouse` | `target` |  |
| `ReferenceHouses[]` | `data[]` |  |
| `Distance`, `Transactiondate`, `Source` | `data[].distance_m`, `data[].transaction_date`, `data[].source` |  |
| `TransactionPrice`, `IndexedTransactionPrice` | `data[].price`, `data[].price_indexed` |  |
| `InnerSurfaceArea`, `BuildYear`, `HouseType` | `data[].living_area_m2`, `data[].construction_year`, `data[].house_type` |  |
| `Weight` | *Not available here* |  |
| `VisualSimilarityScore`, `ReferencePriceMean` | *Not available here* |  |

### Transactions

`POST /v2/transaction` → [`GET /v5/properties/{locator}/transactions`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/transactions)

| Altum | Huispedia | Note |
|---|---|---|
| `valuation_date` | *Not available here* | You get every sale of the home; filter on `transaction_date` yourself. |
| `transaction`, `transaction_date` | `data[].transaction_id`, `data[].transaction_date` |  |
| `transaction_price`, `transaction_price_indexed` | `data[].price`, `data[].price_indexed` |  |
| `buyer`, `seller`, `family_transaction`, `more_real_estate`, `business_right` | *Not available here* |  |

### WOZ value

`POST /woz` → [`GET /v5/properties/{locator}/woz`](https://huispedia.dev/en/docs/endpoints/#tag/properties/GET/v5/properties/{locator}/woz)

| Altum | Huispedia | Note |
|---|---|---|
| `index: 1` | `?index=` |  |
| `wozvalue[].Date`, `Value`, `IndexedValue` | `values[].reference_date`, `values[].value`, `values[].indexed_value` |  |

### Market trend

`GET /market-trend` → [`GET /v5/areas/{area}/stats`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/stats)

| Altum | Huispedia | Note |
|---|---|---|
| `postcode` | `{area}` | Codes from `GET /v5/areas/lookup?postcode=`. |
| `averagePriceSingleFamilyHomes`, `averagePriceMultiFamilyHomes` | `market.avg_transaction_price` | Not split by home type. |
| `pricePerSquareMeterSingleFamily`, `pricePerSquareMeterMultiFamily` | `market.avg_transaction_price_per_m2` |  |
| `annualPriceChange` | `market.transaction_price_per_m2_yoy_pct` |  |
| `transactionData.totalSalesLast12Months` | `market.transactions_count` |  |
| `fiveYearPriceChange` | `data[].value`, `data[].yoy_pct`<br>In [`GET /v5/areas/{area}/trends`](https://huispedia.dev/en/docs/endpoints/#tag/areas-and-market/GET/v5/areas/{area}/trends) | Price per m² per month, five years back by default. |

## Errors

An error is `application/problem+json` with a fixed `code`; every code is on [Errors](https://huispedia.dev/en/docs/errors/).

| Altum | Huispedia | Note |
|---|---|---|
| `400` (address not found) | [`404 not_found`](https://huispedia.dev/en/docs/errors/#not_found) |  |
| `400` (date format, house type), `422` (validation) | [`400 invalid_request`](https://huispedia.dev/en/docs/errors/#invalid_request) | `errors` names what is wrong per field. |
| `400` (no prediction possible) | [`422 data_unavailable`](https://huispedia.dev/en/docs/errors/#data_unavailable) |  |
| `401`, `403` (`Missing Authentication Token`) | [`401 unauthenticated`](https://huispedia.dev/en/docs/errors/#unauthenticated) |  |
| `403` (`Forbidden`, no access to the API) | [`403 entitlement_required`](https://huispedia.dev/en/docs/errors/#entitlement_required) |  |
| `429` (limit) | [`429 rate_limited`](https://huispedia.dev/en/docs/errors/#rate_limited) | Wait `Retry-After` seconds. |
| `429` (quota) | [`402 insufficient_credits`](https://huispedia.dev/en/docs/errors/#insufficient_credits), [`402 spend_cap_reached`](https://huispedia.dev/en/docs/errors/#spend_cap_reached) |  |
| `500` | [`500 internal_error`](https://huispedia.dev/en/docs/errors/#internal_error) |  |
| `{"Output": "…"}`, `{"detail": […]}` | `application/problem+json` with `code`, `detail`, `errors` |  |

## Bulk and limits

|  | Altum | Huispedia |
|---|---|---|
| **Rate** | Fair use: up to 6 concurrent requests per second | [6 per second and 6 at once on a paid plan; 1 and 2 on Free](https://huispedia.dev/en/docs/rate-limits/) |
| **Bulk** | `POST /avm/bulk`, then fetch the job | `POST /v5/valuations/scenarios`: 100 homes per call, answered at once |
| **Usage** | `GET /credit` | `GET /v5/account/usage` |

## Example: home valuation

**Altum**

```bash
curl -X POST https://api.altum.ai/avm \
  -H "x-api-key: $ALTUM_KEY" \
  -H "Content-Type: application/json" \
  -d '{"postcode": "1076LJ", "housenumber": 12, "houseaddition": "2"}'
```

```python
import os, requests

r = requests.post(
    "https://api.altum.ai/avm",
    headers={"x-api-key": os.environ["ALTUM_KEY"]},
    json={"postcode": "1076LJ", "housenumber": 12, "houseaddition": "2"},
)
value = int(r.json()["Output"]["PriceEstimation"])
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

- Photo with the valuation (`image`): not available here.
- Pointing at a home by verblijfsobject id: not available here; use the nummeraanduiding.
- A valuation more than six months back: not available here; the valuation history gives one value per month.
- Your own weights, a weight per reference home and visual similarity: not available here; the order gives the likeness.
- Buyer, seller and Kadaster codes with transactions: not available here.
- Municipal charges (`ozb_tax` and the like): not available here.
- Average WOZ per postcode (`/woz/pc6`): not available here.
- Rental comparables, WWS points, rebuild and contents value: not available here.
- NTA 8800 label calculation, sustainability, subsidies, roof scans, EPC: not available here; `GET /v5/properties/{locator}/energy-label` gives the registered label.
- Move data, building geometry, photo labelling: not available here.

### No equivalent at Altum

- `409 address_ambiguous` with the candidates when an address has several homes: no equivalent at Altum.
- What-if scenarios against the same base (`scenarios`): no equivalent at Altum.
- Monthly valuation history in one call: no equivalent at Altum.
- Environment (foundation, subsidence, noise, air) and amenities per address: no equivalent in Altum's public documentation.
- A building with all its addresses (`/v5/buildings/{bag_pand_id}`): no equivalent in Altum's public documentation.

## Sources

- [Woningwaarde API](https://docs.altum.ai/taxeren-en-waarderen/woningwaarde-api/authentication-input-and-response)
- [Woningwaarde API, huisnummers en toevoegingen](https://docs.altum.ai/taxeren-en-waarderen/woningwaarde-api/house-numbers-and-additions)
- [Woningwaarde API, accuratesse-indicator](https://docs.altum.ai/taxeren-en-waarderen/woningwaarde-api/accuratesse-indicator)
- [Woningwaarde API, OpenAPI](https://api.altum.ai/avm/openapi)
- [Woningwaarde+ API](https://docs.altum.ai/taxeren-en-waarderen/woningwaarde+-api/authenticatie-invoer-en-resultaat)
- [Woningkenmerken API](https://docs.altum.ai/woningdata/woningkenmerken-api/authenticatie-invoer-en-resultaat)
- [Interactieve woningreferentie API](https://docs.altum.ai/taxeren-en-waarderen/interactieve-woningreferentie-api/authenticatie-invoer-en-resultaat)
- [Kadaster Transactie API](https://docs.altum.ai/taxeren-en-waarderen/kadaster-transactie-api/authenticatie-invoer-en-resultaat)
- [WOZ API](https://docs.altum.ai/woningdata/woz-api/authentication-input-and-response)
- [Markttrend API](https://docs.altum.ai/taxeren-en-waarderen/markttrend-api/authentication-input-and-response)
- [Sandbox](https://docs.altum.ai/ontwikkelaars/sandbox)
- [Fair use policy](https://docs.altum.ai/enterprise/fair-use-policy)
- [Foutcodes 403, 422, 429](https://docs.altum.ai/api-error-codes/422)
- [Verbruik API](https://docs.altum.ai/ontwikkelaars/verbruik-api)
