# OSEP #7: Restoring Events Data

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Final    |
| **Issue**          | https://github.com/openstates/enhancement-proposals/issues/34 |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-08-13 |
| **Updated**        | 2021-10-13 |

---

## Abstract

From around 2010-2015 Open States scraped "events" data, mostly data on upcoming hearings and legislative meetings.  That data has not been part of Open States' data offerings for quite a while, despite maintained scrapers in many states.

This proposal would restore events data to Open States' public data offerings.

## Specification

The existing events scrapers would regain "first class" status, and be run regularly (daily at least).

The existing schema would be left mostly intact, and Event importers would be restored.  Additional scrape jobs would be configured to run event scrapers regularly again & report failures similarly to how bill scrapers are configured today.

API v3 will be updated to include event data.   For now, OpenStates.org & API v2 will not be affected.

Some changes will be introduced as part of this proposal:

### Schema Changes

Scraped Events will gain the following optional fields:

- `upstream_id`: This can be used to record an upstream identifier, such as a database identifier that can be obtained from the source data.  If present it will be used to uniquely identify events.  (This is distinct from dedupe_key which has no semantic meaning and can be a constructed value or URL.)

### Soft Deletes on Import

To better address downstream user's needs, when future events can not be found in the current scrape nor reconciled via the standard means (`dedupe_key`, the new `upstream_id`, nor a match on the main attributes), the future event will be marked as `deleted` in the database.

Deleted events will not be returned in the API response by default, but may be explicitly requested by clients.

### Windowing

In some states we have to select a date range to scrape, which currently leads to ad hoc code (as described by Tim here: https://github.com/openstates/enhancement-proposals/pull/28#issuecomment-898720989)

To bring some sense of standardization to this, we will standardize on the following parameters:

- `today` (default: today's date, can be overridden for testing purposes)
- `days_before` (default: 30)
- `days_after` (default: 90)

All of which can be overridden on the command line.  A scraper that supports this interface will call a utility function with 
these variables and obtain a `start_date` & `end_date` centered around the `today` value.  The obtained start/end dates should then be used by the scraper to window its request to the source.

### Helper Methods

The `openstates.scrape.Event` object will gain a couple of helper functions:

* `Event.add_bill` which will add an agenda item with configurable placeholder text (if one does not exist already) and associate a bill with that item.  This will serve as a workaround to associate bills with events that do not have an appropriate agenda item already.
* `Event.add_media_link` will gain a `classification` parameter to classify the type of media being added.  Options will include: 'video', 'audio', 'hosted video', 'hosted audio', 'youtube'.


## Rationale

This is valuable data, and with renewed development resources, it has again become feasible to support maintaining it.

## Drawbacks

The main drawback here is just in terms of resource allocation, supporting this across states will be significant work.  There are no backwards compatibility concerns to address.

## Implementation Plan

Tim has already been maintaining the events scrapers within the openstates-scrapers repository.  Now they will be maintained by other members of the team as well.

The additional work to support the import of events & access via the API will be done by James or others under his supervision.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)

