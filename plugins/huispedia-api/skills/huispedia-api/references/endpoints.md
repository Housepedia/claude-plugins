# Endpoints

> Generated from https://huispedia.dev/openapi.json. Credits per successful call; base URL https://huispedia.dev.

## Catalogue

### `GET /v5/catalog`

The catalogue (free). The credits per endpoint, the free tier, the limits and the sources. No key needed.

## Account

### `GET /v5/account`

Your account (free). The balance by bucket, the spend cap, the limits and the entitlements of the account this key belongs to.

Response: `account_id`, `type`, `name`, `locale`, `plan`, `balance`, `spend_cap`, `limits`, `entitlements`, `terms_version`, `meta`

### `GET /v5/account/usage`

Usage per day, endpoint and key (free). Calls and credits per day, endpoint or key, at most 92 days per call.

Parameters:
- `from` (query): Start of the period.
- `to` (query): End of the period.
- `group_by` (query): Comma list of day, endpoint, key.

Response: `data`, `meta`

### `GET /v5/account/requests`

The request log (30 days) (free). One row per call of the last 30 days: when, which endpoint, status, credits and duration.

Parameters:
- `from` (query): Start of the period.
- `to` (query): End of the period.
- `key_id` (query): Only calls with this key.
- `status` (query): Only calls with this HTTP status.
- `limit` (query): Rows per page, at most 100.
- `cursor` (query): The next page: meta.next_cursor of the previous answer.

Response: `data`, `meta`

## Addresses

### `GET /v5/addresses/suggest`

Address autocomplete (free) (free). Finds addresses and streets while you type: by postcode, postcode plus number, street, or street plus number and city. Use the locator of a result on the other endpoints.

Parameters:
- `q` (query, required): Postcode, postcode + number, street, or street + number[, city].
- `limit` (query): At most this many results.
- `types` (query): Comma list of address, street, postcode (all three by default).

Response: `data`, `meta`

### `GET /v5/addresses/reverse`

Reverse geocoding (1 credits). The nearest addresses to a point, nearest first, and the areas the point lies in.

Parameters:
- `lat` (query, required): Latitude, WGS84.
- `lon` (query, required): Longitude, WGS84.
- `radius_m` (query): Search radius in metres.
- `limit` (query): At most this many addresses.

Response: `data`, `areas`, `meta`

### `GET /v5/addresses/{locator}`

Address validation (1 credits). Checks an address and returns it in its standard form, with its BAG ids and the areas it lies in. `409 address_ambiguous` lists the homes at a postcode plus number when no extension is given; `all=true` returns them all instead.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `all` (query): Return every home at the address instead of a 409 when the address is ambiguous.
- `include` (query): Comma list of the extras this endpoint offers, such as rd.

Response: `bag_id`, `postcode`, `house_number`, `house_number_ext`, `street`, `city`, `municipality`, `province`, `location`, `rd`, `huispedia_url`, `bag_verblijfsobject_id`, `bag_pand_id`, `bag_status`, `is_residential`, `neighbourhood`, `district`, `municipality_code`, `province_code`, `meta`, `data`

## Properties

### `GET /v5/properties/{locator}`

Property characteristics (2 credits). Up to sixty fields about the home, in groups. `fields` picks groups. The sales of the home are under Transactions.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `fields` (query): Comma list of the groups to return: type, dimensions, rooms, outdoor, construction, energy, ownership, maintenance. All of them by default.
- `include` (query): Comma list of the extras this endpoint offers, such as rd.

Response: `address`, `type`, `dimensions`, `rooms`, `outdoor`, `construction`, `energy`, `ownership`, `maintenance`, `meta`

### `GET /v5/properties/{locator}/energy-label`

Energy label (1 credits). The registered energy label with its recording and validity dates, and earlier labels. A label from a listing when none is registered.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

Response: `address`, `label`, `source`, `recording_date`, `valid_to`, `registration_date`, `is_expired`, `history`, `meta`

### `GET /v5/properties/{locator}/woz`

WOZ value and history (2 credits). The WOZ value per reference date since 2015, newest first. `index=true` adds each value at today's price level.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `index` (query): Also give each value at today's price level (indexed_value).

Response: `address`, `latest`, `values`, `meta`

### `GET /v5/properties/{locator}/condition`

Condition score (2 credits). The maintenance state of the home as assessed from its listing photos: overall, modernity and finish on a scale of 1 to 5, with confidence and a short reasoning.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

Response: `address`, `available`, `source`, `assessed_at`, `overall`, `modernity`, `finish`, `style`, `reasoning`, `meta`

### `HEAD /v5/properties/{locator}/transactions`

Transactions available? (free) (free, with contract). 200 when the home has transactions, 404 when it has none.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

### `GET /v5/properties/{locator}/transactions`

Transactions (2 credits, with contract). The sales of the home with their prices, and the listings they came from. Available with a contract; contact sales@huispedia.nl. `X-Kadaster-Records` counts the Kadaster records in the answer.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

Response: `address`, `data`, `meta`

### `GET /v5/buildings/{bag_pand_id}`

Building and its units (1 credits). Every address in a BAG building: house numbers, floor, living area and whether it is a home, with the building's construction year and location.

Parameters:
- `bag_pand_id` (path, required): The 16-digit BAG id of the building.

Response: `bag_pand_id`, `construction_year`, `bag_status`, `units_count`, `residential_units_count`, `units`, `location`, `street`, `postcode`, `city`, `meta`

## Valuations

### `GET /v5/properties/{locator}/valuation`

Home valuation (8 credits). The current Huispedia home value: value, 80% interval and confidence. `as_of_date` gives the value at an earlier month, up to six months back.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `as_of_date` (query): A month up to six months back; the value at that month's price level.

Response: `address`, `valuation`, `confidence`, `meta`

### `GET /v5/properties/{locator}/references`

Reference homes (8 credits, with contract). Comparable homes sold nearby, ranked by similarity, distance or date, within the bounds you set. Kadaster sale prices come with a contract.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `limit` (query): Number of reference homes, 3 to 30.
- `max_distance_m` (query): No further away than this, in metres.
- `max_age_months` (query): Sold no longer ago than this many months.
- `same_type` (query): Only homes of the same type (house or apartment).
- `min_living_area_m2` (query): Smallest living area, m².
- `max_living_area_m2` (query): Largest living area, m².
- `min_construction_year` (query): Earliest construction year.
- `max_construction_year` (query): Latest construction year.
- `as_of_date` (query): The date to look back from; today by default.
- `sort` (query): similarity, distance or date.

Response: `target`, `data`, `meta`

### `GET /v5/properties/{locator}/valuation/plus`

Home valuation plus (12 credits, with contract). The valuation with the five best reference homes, the neighbourhood's market figures and the home's latest WOZ value, in one call.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `as_of_date` (query): A month up to six months back; the value at that month's price level.

Response: `address`, `valuation`, `confidence`, `meta`, `references`, `neighbourhood`, `woz`

### `GET /v5/properties/{locator}/valuation/history`

Valuation history (3 credits). One value per month with its interval, 36 months back by default. `current` carries this month's value when it is newer than the last monthly point. `include=changes` adds what changed about the home between points.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).
- `from` (query): First month; 36 months back by default.
- `to` (query): Last month; this month by default.
- `include` (query): Comma list of the extras this endpoint offers, such as rd.

Response: `address`, `data`, `current`, `meta`

### `POST /v5/valuations/scenarios`

Scenario valuation (20 credits). Values up to 100 homes with the characteristics you give, and each named scenario against the same base. Each row and each scenario is a valuation of its own; a row that fails carries `error`.

Parameters:
- body: `data`

Response: `data`, `meta`

## Areas and market

### `GET /v5/areas/lookup`

Area lookup (1 credits). The neighbourhood, district, municipality and province around a point or a postcode.

Parameters:
- `lat` (query): Latitude, WGS84.
- `lon` (query): Longitude, WGS84.
- `postcode` (query): A full postcode, such as 1076LJ.

Response: `neighbourhood`, `district`, `municipality`, `province`, `meta`

### `GET /v5/areas/{area}`

Area (1 credits). An area's name, codes, parents and centre; `include=geometry` adds its outline as GeoJSON.

Parameters:
- `area` (path, required): level:code, such as neighbourhood:BU0363KH05, municipality:GM0363, postcode4:1076 or national.
- `include` (query): Comma list of the extras this endpoint offers, such as rd.

Response: `type`, `code`, `huispedia_id`, `name`, `slug`, `parents`, `centroid`, `cbs_vintage_year`, `geometry`, `meta`

### `GET /v5/areas/{area}/stats`

Area statistics (3 credits). Market, housing stock and environment figures for the area over the last twelve months. A larger area combines its neighbourhoods.

Parameters:
- `area` (path, required): level:code, such as neighbourhood:BU0363KH05, municipality:GM0363, postcode4:1076 or national.
- `period` (query): last_12_months.

Response: `area`, `type`, `period`, `market`, `stock`, `environment`, `meta`

### `GET /v5/areas/{area}/trends`

Price index (2 credits). The price per m² per month with its change against a year earlier, five years back by default. `include=parents` adds the district, municipality, province and national series for a neighbourhood.

Parameters:
- `area` (path, required): level:code, such as neighbourhood:BU0363KH05, municipality:GM0363, postcode4:1076 or national.
- `metric` (query): transaction_price_per_m2 or asking_price_per_m2.
- `interval` (query): month.
- `from` (query): First month; five years back by default.
- `to` (query): Last month; this month by default.
- `include` (query): Comma list of the extras this endpoint offers, such as rd.

Response: `area`, `type`, `metric`, `interval`, `data`, `parents`, `meta`

### `GET /v5/areas/{area}/demographics`

Demographics (2 credits). Population, households and age groups of a neighbourhood, from CBS.

Parameters:
- `area` (path, required): level:code, such as neighbourhood:BU0363KH05, municipality:GM0363, postcode4:1076 or national.

Response: `area`, `population`, `population_density`, `address_density`, `households`, `age`, `cars_per_household`, `cbs_vintage_year`, `meta`

### `GET /v5/properties/{locator}/surroundings`

Surroundings (2 credits). The nearest amenity per category (supermarket, school, GP, station and more) with its distance, and the counts in the neighbourhood.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

Response: `address`, `nearest`, `counts_in_neighbourhood`, `mobility`, `meta`

### `GET /v5/properties/{locator}/environment`

Environment (3 credits). Foundation risk, subsidence, noise, air quality, heat and light pollution at the address, with the source of each.

Parameters:
- `locator` (path, required): `bag:<nummeraanduiding>` (`bag:0363200000028422`) or `<postcode>-<number>[-<extension>]` (`1076LJ-12-2`).

Response: `address`, `foundation`, `subsidence`, `noise`, `air`, `heat`, `light_pollution`, `environmental_health_risk`, `bee_richness`, `sources`, `meta`
