# OSEP #6: New People Offices Schema

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Draft    |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/27 |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-08-13 |
| **Updated**        | TODO |

---

## Abstract

Make improvements to how office/contact information is represented to allow better modeling of real world scenarios & reduce user confusion.

## Specification

Currently Open States stores contact information in a highly normalized form:
**ContactDetail**
- type (address|email|url|fax|text|voice|video|pager|textphone) [1]
- value
- note
- person_id

This is based on the _Popolo_ spec, which heavily influenced Open Civic Data's modeling.  One reason for this was to avoid a bunch of null columns for lesser used values, but in practice, we only use `voice`, `fax`, and `address`.  (`email` was used but then moved to a top level field in early 2020).

In practice though, we typically tie addresses together "e.g. Capitol Office or District Office".  This is done via `note`, which is now required to be either `Capitol Office`, `District Office`, or `Primary Office`.

This proposal suggests moving us towards a model that more accurately represents how most states provide (and users use) the data:

**Office**
- classification: capitol | district | primary
- address
- voice
- fax
- name (optional)

With additional validation rules:
- an office *must* have at least one of `address`, `voice`, `fax`.
- At most one address may have the classification `capitol` or `primary`.
- Any number of `district` addresses may be provided.
- If `name` is not provided, an office will display as `Capitol Office`, `District Office`, or `Primary Office` as it does now (based on `classification`)

## Rationale

As noted above, we currently group address information into Capitol Office & District Office (& occasionally Primary Office).  Doing so does not allow us to distinguish between multiple district offices, which are common in certain states (e.g. California).  This leads to unclear pairings of data on OpenStates.org as well as an issue in API v3 which attempts to do this grouping for user convenience.

Additionally, the code to merge scraped people is quite complex and has several conditions that cause it to bail & request manual resolution.  These conditions are direct consequences of the current schema.  

For more clear examples of the issues presented, assume a person has the following ContactDetail records:
- type=voice, note=District, value=555-555-1234
- type=address, note=District, value=123 Main Street
- type=voice, note=District, value=555-555-6666
- type=address, note=District, value=1600 Pennsylvania Ave

OpenStates.org will show two district offices, but there is no way to know which number corresponds to which address, so they will be grouped based on database retrieval order.

API v3 will only show one district office, since the logic there is to attempt to convert ContactDetail records into more usable "offices" which works well for most cases, but the pairing information is not currently available in this case.

Finally, the people merge code (openstates-core/people) will not be able to merge any changes that affect the District offices since it can not be confident which phone number was added/removed/changed.  (Essentially due to missing a unique data path to the changed item, which is not an issue with any other piece of data at present.)

## Drawbacks

We will lose the flexibility of supporting other contact detail types.  Given that they haven't been used thus far makes that feel like a reasonable trade-off.

This proposal will require consumers of the raw people data to update their code, as the `openstates/people` repository will need to be updated, with a migration script run to convert existing offices to the new format.

## Implementation Plan

After approval, a script will be written to convert `openstates/people` to the new data format.  All appropriate data scripts (`os-people to-database` and `merge`) will be updated, initially to import the data to both the old `ContactDetail` table as well as the new `Offices` table.

Initially this means that upstream impact will be limited, this will give us time to update OpenStates.org, API v2, and v3.

OpenStates.org's display logic will be changed to support the new `offices` data.

From this point, the GraphQL API will be updated to support both forms with a transformation step to allow backwards compatibility.  The new `offices` key will be added and queryable via GraphQL, whereas the old `contactDetails` key can be left intact as-is with a backwards-compatibility shim put in place.

API v3 will be updated to query the database's `offices` key.  Given that it already uses the `offices` key in responses, the new `name` & `classification` key will be the only impact there.

After all of these changes are implemented, the old `ContactDetail` table can be dropped from the database & import process.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
