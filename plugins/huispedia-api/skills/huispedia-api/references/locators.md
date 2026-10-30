# Addresses and areas

> Pointing at a home or an area in the path.

Source: https://huispedia.dev/en/docs/locators/

## Homes

A home appears in the path as `:locator`, in one of two forms:

| Form | Example | What you pass |
|---|---|---|
| BAG nummeraanduiding | `bag:0363200000028422` | The 16-digit BAG id of the address |
| Postcode address | `1076LJ-12-2` | Postcode, house number and the extension when there is one; case and spaces do not matter |

Don't know the extension? `GET /v5/addresses/suggest?q=Amazonenstraat 12, Amsterdam` looks the address up (free); `GET /v5/addresses/1076LJ-12?all=true` returns every home at a house number.

In a POST body (scenario valuations) you identify a home with `locator`, `bag_id` or `postcode` + `house_number` (+ `house_number_ext`).

## Areas

An area appears in the path as `:area`, always as `level:code`:

| Level | Code | Example |
|---|---|---|
| `neighbourhood` | CBS neighbourhood code | `neighbourhood:BU0363KH05` |
| `district` | CBS district code | `district:WK0363KH` |
| `municipality` | CBS municipality code | `municipality:GM0363` |
| `province` | CBS province code | `province:PV27` |
| `postcode4` | The four digits of the postcode | `postcode4:1076` |
| `postcode6` | The full postcode | `postcode6:1076LJ` |
| `national` | No code | `national` |

The CBS codes are in every answer about an address, or look them up with `GET /v5/areas/lookup?postcode=1076LJ`. Statistics, the price index and demographics exist per neighbourhood, district, municipality, province and the whole country.
