# OSEP #12: Improved Entity Matching

|                    |                                                                |
|--------------------|----------------------------------------------------------------|
| **Author(s)**      | Rylie                                                          |
| **Implementer(s)** | Rylie                                                          |
| **Status**         | Draft                                                          |
| **Issue**          | https://github.com/openstates/enhancement-proposals/issues/TBD |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/TBD   |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD   |
| **Created**        | 2024-07-01                                                     |
| **Updated**        | TODO                                                           |

---

## Abstract

With the 2024 New Session, we had far more eyes on Events & Votes as well as our usual Bill activity. Working through
bug tickets, it became evident that there was only so much we could do for some scrapers but some missing data could be
traced back to lack of proper matching. This EP is to start improving the matching by passing in data that would narrow
the query results returned on import.

## Specification

TODO: Describe how the proposal will work.

To help resolve People mismatching, there is already an option to pass in an `org_classification` to the
[resolve_person](https://github.com/openstates/openstates-core/blob/ac8e53aefe2a70d8ff360fc8b641bf77f28e2d7c/openstates/importers/base.py#L526)
function on the `BaseImporter` that is used to query & match People to Bills, Events, & Votes. If the
`org_classification` isn't set, it just defaults to a combination of `upper`, `lower`, & `legislature`. If we ensure
that an `org_classification` can be passed in from where it's used in the Bill, Event, & Vote importers, we should be
able to alleviate some of that mismatching. There may need to be some scraper updates to ensure that the classification
is correct, like a Bill getting sponsors added from the opposite chamber than it was introduced in, but for Votes where
the voting body is either a Chamber or a Committee, we can narrow down People by classification based off of that voting
body with more accuracy.

Similarly, in helping resolve Committees, we can improve the matching query by cleaning or splitting up the scraped name
into it's different Committee elements such as Chamber & Type and then incorporating that into the `OrganizationImporter`
[limit_spec](https://github.com/openstates/openstates-core/blob/ac8e53aefe2a70d8ff360fc8b641bf77f28e2d7c/openstates/importers/organizations.py#L11)
logic. This will be a bit messier, but we could also add `other_names` to Committee files to more easily match up against
what is commonly scraped like we did [for MN](https://github.com/openstates/people/pull/1442/files) when Events were
"missing" because of name mismatching & update the `limit_spec` logic to check for more than the first `other_name`
string.

In resolving Committees as Bill Sponsors, there's logic that should be able to match in the `BillImporter`'s
[prepare_for_db](https://github.com/openstates/openstates-core/blob/ac8e53aefe2a70d8ff360fc8b641bf77f28e2d7c/openstates/importers/bills.py#L147)
function, so need to ensure that scrapers are checking if the Sponsor is a Person or Organization & make sure that is
being correctly passed in as the `entity_type` in `add_sponsorship()`.

When it comes to matching Bills to Agenda Items on Events, I'm a little more fuzzy. Right now we have a [resolve_bill](https://github.com/openstates/openstates-core/blob/ac8e53aefe2a70d8ff360fc8b641bf77f28e2d7c/openstates/importers/base.py#L164)
function on the `BaseImporter` that attempts to match Bills via `bill_id`, `jurisdiction_id`, & `date` if it gets passed,
which seems like it could be improved by incorporating some of the logic in `resolve_related_bills` that Jesse worked on
this spring where the match query is also narrowed down by `session_id`.


## Rationale

TODO: Explain the reason for this in detail.  Discuss alternatives considered.

We've known that matching Bills or Votes to Sponsors has been tricky for a while, hence OSEP #3 to help alleviate some
of the issues with mismatching legislators. The People Matcher Tool can only get us so far, since we run into a blocker
when there are legislators with the same last name in a jurisdiction or the sponsor is actually a committee, where
adding an `other_name` to a person's yaml file isn't a possible fix.

A similar issue has been happening with matching Events to their Participants (typically a Committee). The scraped name
of a participant can vary from vague things such as "Rules" with no chamber, or more specific like "Assembly Privacy and
Consumer Protection Committee" but name of the Committee doesn't have the chamber listed on the yaml file. Now that
we've come to a standard expectation for the OS People repo that Committees will just be the name without chamber &
committee type since those are able to be derived from data in the yaml file, this should make it easier to match with
if we can narrow the match query based on those attributes.

Another area where we're struggling to match entities is Events to the Bills listed in their Agenda Items. Sometimes
it's clearly because the scraped bill id format is different from how the Bill gets saved, but sometimes it's less clear
as to why some Bills get matched but others don't. Occasionally, there may be a Bill that doesn't exist in OS yet but
is mentioned as an Event's Agenda Item, so it won't be attached to the Event until after a future scrape after the Bill
is in the system. 

## Drawbacks

Should absolutely add defaults if we're not certain what's going to be passed in.

## Implementation Plan

TODO: How will this be done?  Are you volunteering to do it?  Do you want someone else to do it?
(It is ok to leave this blank in a draft if you aren't sure.)

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
