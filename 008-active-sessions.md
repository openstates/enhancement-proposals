# OSEP #8: Active Sessions

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Final    |
| **Issue**          | https://github.com/openstates/enhancement-proposals/issues/29 |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/30 |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-08-16 |
| **Updated**        | 2021-11-08 |

---

## Abstract

This proposal would add a flag to sessions indicating whether or not it is considered active or not.  This information would be used for determining which sessions to scrape by default, and could be used within the openstates.org UI as well.

## Specification

The Legislative Session schema will be updated to have a required field named `active` to indicate if the session is currently receiving updates.  Other portions of the code will be updated to use this field as needed, most notably the code that selects which session to run if no session is given on the command line, and the OpenStates.org search logic.  This field will also be surfaced in the APIs so that downstream consumers can make decisions based upon it as they wish.

Additional care will be taken to add validation rules that either error out or warn loudly if the flag is potentially set incorrectly.

## Rationale

This helps address two major problems that are present right now:
- When scrapers are run, only the latest (by order of appearance in the metadata) session is scraped.  Due to complex special session rules, that means that the active session information winds up within the task-definitions repository, where we add additional tasks for one-off re-scrapes, special sessions, etc.  This also means when running locally, special care must be taken to run with the correct session=XYZ argument.
- When doing things like full text search, that should default to the current session- we use the same rule (latest in the metadata) whereas it might be more beneficial to search within multiple sessions or a session that is not the latest but is more likely to contain the bill(s) being searched for.

The main alternative considered was the idea to use begin & end dates instead (see https://github.com/openstates/enhancement-proposals/issues/29).  It was pointed out that there are a lot of special cases which would render this solution ineffective, for example:
- IL governor sometimes signs bills from the previous session even after the new one started, this year I think that happened as late as march 2022 after 2021 regular session close.
- VA as of August 2021 is in a state where the Regular session is getting updates (from the executive) despite being adjourned.  Special #1 is finished, and Special #2 is actively meeting.
- Prefiles will need to be considered active before their start date to be scraped.

These cases make using start & end date fraught, as we'd wind up needing to manipulate them to our needs or still falling back to overrides.

## Drawbacks

This will be an extra piece of data to manually maintain across each state.  It is somewhat redundant with the begin & end date (but not entirely, as noted above in Rationale).

It is not unlikely that we'll run into issues where we forget to update the flag & continue to scrape old sessions.  As noted in the specification, special care will be taken to try to add checks for this behavior that run alongside the session check code that runs at scraper start.

## Implementation Plan

James will take the lead on adding this to openstates-core:
- schema support for LegislativeSession.active
- expose & document new flag in APIs
- additional checks at scraper start that the active session list is sane
- an automated update across all jurisdictions to set active sessions (defaulting to use current last-is-active logic)
- fix `os-update` command to use new logic
- fix OpenStates.org search to use active sessions by default instead of current logic which assesses latest session with any bills

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
