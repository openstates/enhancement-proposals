# OSEP #5: Replacing pupa_id with dedupe_key

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Draft    |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/19 |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-04-01 |
| **Updated**        | 2021-04-01 |

---

## Abstract

There are cases where the automated import pipeline cannot distinguish between two similar objects and relies upon a hint from the scraper to decide if an object should be treated as unique or not.
Right now this is done by setting pupa_id, which is then stored in the pupa_identifier table with a join to the object in question.
This logic is not well understood, and the naming was poorly thought out.
This is a small OSEP that will fix this, making other deduplication work in the future easier to perform.

## Specification

Scraped VoteEvents will gain a new field 'dedupe_key' which will be optional and function identically to pupa_id on import.

## Rationale

The current name was never great, and as the last real reference to pupa makes it even more confusing to new people that aren't aware of the history.

Fixing this code now will make other import-side improvements to come easier.

## Drawbacks

Very few, a little disruptive to alter a bunch of scrapers at once, but the plan allows doing it incrementally if needed.

This will be a minor change to the scrape output, which is the main reason this rises to the level of OSEP.

This will not have any public facing impact.

## Implementation Plan

This needs to be done in several steps to avoid disruption:

- addition of VoteEvent.dedupe_key in database
- write a migration script that copies data from the pupa_identifier table to VoteEvent.dedupe_key and drops the old pupa_identifier table
- write updated version of openstates-core which supports pupa_id in the scrape output but saves to dedupe_key on the backend
- when ready, near-simultaneous release of openstates-core and run of migration script
- update scrapers to not use pupa_id anymore
- once all scrapers updated, release updated version of openstates-core which no longer supports pupa_id in the scrape output

James will handle all of this, once approved it should take 1-2 hours.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
