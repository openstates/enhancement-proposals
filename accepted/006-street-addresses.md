# OSEP #6: Structured data for street addresses

|                    |            |
|--------------------|------------|
| **Author(s)**      | @paulschreiber |
| **Implementer(s)** | TODO |
| **Status**         | Draft   |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/18 |
| **Approval PR(s)** | 
| **Created**        | TODO |
| **Updated**        | TODO |

---

## Abstract

Open States provided currently provides a single field for address data. It would be beneficial to have separate street, city, state and zip code fields.

Within the current `address` field, the information available, spacing and delimiters vary by state, making parsing difficult.

## Specification

### Data Model Changes

- Under `contact` details, for both district and capitol offices, add the fields `street`, `city`, `state` and `zip`
- For backwards compatibility, existing `address` field will be preserved (temporarily).

### Scraping

Data availability and format varies by state 
* some states provide the data in separate fields
* some states do not provide the state name
* some states inconsistently uses state names and state abbreviations

Observed cases can easily be parsed with regular expressions.

## Rationale

This would allow for easier searching, sorting, geocoding and other data uses.

## Drawbacks

- In the short term, duplicate data is stored.

## Implementation Plan

I will assist with updating scrapers to output the structured data.

The address field will be a composite of the structured data (using format strings).

Several team members, and hopefully some community members, will help to contribute updated committee scrapers.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
