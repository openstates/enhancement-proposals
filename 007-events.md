# OSEP #7: Restoring Events Data

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Draft    |
| **Issue**          | https://github.com/openstates/enhancement-proposals/issues/34 |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-08-13 |
| **Updated**        | TODO |

---

## Abstract

From around 2010-2015 Open States scraped "events" data, mostly data on upcoming hearings and legislative meetings.  That data has not been part of Open States' data offerings for quite a while, despite maintained scrapers in many states.

This proposal would restore events data to Open States' public data offerings.

## Specification

The existing events scrapers would regain "first class" status, and be run regularly (daily at least).

The existing schema would be left intact, and Event importers would be restored.  Additional scrape jobs would be configured to run event scrapers regularly again & report failures similarly to how bill scrapers are configured today.

API v3 will be updated to include event data.   For now, OpenStates.org & API v2 will not be affected.

## Rationale

This is valuable data, and with renewed development resources, it has again become feasible to support maintaining it.

## Drawbacks

The main drawback here is just in terms of resource allocation, supporting this across states will be significant work.  There are no backwards compatibility concerns to address.

## Implementation Plan

Tim has already been maintaining the events scrapers within the openstates-scrapers repository.  Now they will be maintained by other members of the team as well.

The additional work to support the import of events & access via the API will be done by James or others under his supervision.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)

