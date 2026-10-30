# Versioning

> What may change within v5, and what never does.

Source: https://huispedia.dev/en/docs/versioning/

Within v5 every change is an addition:

- new endpoints;
- new optional parameters and request fields;
- new fields in an answer;
- new values in a list of values, such as a new `house_type` or error `code`.

Nothing is removed, renamed, made required or made stricter. Code that works today keeps working, provided it ignores fields it does not know and accepts values it has not seen.

A change that would break that comes only in a new major version (v6). We announce it at least 60 days ahead, by email and in the [changelog](https://huispedia.dev/en/docs/changelog/) ([API terms](https://huispedia.dev/en/voorwaarden/), art. 11.2). v5 keeps running next to it for a period we state there.
