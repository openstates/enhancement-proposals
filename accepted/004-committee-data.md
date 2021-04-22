# OSEP #4: Committee Data

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | TODO |
| **Status**         | Accepted   |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/18 |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-03-31 |
| **Updated**        | 2021-04-22 |

---

## Abstract

Open States provided committee membership data from 2011-2018, but ceased to when it was no longer viable given the project's funding and staffing situations.  This proposal would restore committee membership data via a hybrid scraped/manual process with a focus on maintaining up-to-date current membership of committees.

## Specification

### Data Model Changes

The data model Open States inherited from OCD/Popolo is still a part of the core OS models and will be renewed to support this work.

As part of this proposal, several small changes will be made to make working with this data slightly easier:

- `OrganizationLink` and `OrganizationSource` will be migrated to inline JSON fields on the `Organization` model.
- `Organization` will also gain an `other_names` JSON field.

These changes can be implemented in a way that does not impact any of Open States' public APIs.


### Scraping

Right now we have 45 committee scrapers in openstates-scrapers that have not been run in years.  A quick check of 10 of them showed that the vast majority do not run as-is.

That fact combined with the limitations in the old pupa import path for Organizations points towards us adopting a different method that allows easier augmentation of scraped data with manual.  To this end, we will port/rewrite committee scrapers to a new format similar to the direction that new people scrapers are heading.

The proposed path looks like this:

- The committee portions of openstates-people will be restored:
	- each jurisdiction will again have a committees directory
	- lint_yaml.py will be renamed lint_people.py
	- a new lint_committees.py will be created
	- the committee portion of to_database.py will be restored
	- the committee portion of to_csv.py will be restored
	- documentation and CI processes will be updated to include committee data
- Committee scrapers will be added to `openstates-people` and be written in a way that yields compatible YAML.
	- Existing committee scrapers will be kept in openstates-scrapers until that state has been ported to openstates-people.
	- new committee scrapers will be run regularly with changes to the YAML submitted as PRs (manually at first, but likely the subject of future automation)

### Manual Data

The existing organization schema will be updated to require either ID or name, if ID is present it will be used for linking, whereas name will serve as a placeholder for unlinked IDs.

Initially we'll be focused on manual data that is not impacted by the scrapers, such as alternate names and links.  As such the tool that merges scraper output with the YAML will be focused on replacing member names with those found by the most recent scrape.

This also makes it possible to add committee data by alternate processes, particularly useful in cases where scraping a given chamber or state's committees is harder than maintaining a manual roster.

We will not focus on historical data, intending the committee membership in the file to reflect the membership at a given time.  It is possible that in the future we could use git history to try to construct historical records from this.

This manual data process also leaves the door open for future iterations where some data could be marked in the YAML as to-be-kept so that the membership of a committee could be partially manually curated, but that will be kept to future enhancements.

### API v2

API v2 will be updated in a backwards-compatible way, such that the existing organizations node can be used to fetch committee data again.

### API v3

Committees search & detail endpoints will be added to API v3 in the same manner as existing Jurisdictions/People endpoints.

## Rationale

Restoration of committees has been one of the most requested data sets since their deprecation was deemed necessary.  Now that there are resources available to pursue this again it is a high priority addition to our existing data.

As for the approach itself, it mainly draws on the experience of the earlier iterations of committee data.  Drawing upon the expertise gained in the first iteration, we are not attempting a fully automated process as augmentation & deduplication became quite difficult under that approach.  The manual approach on the other hand quickly went stale and was not updated in any meaningful way after the initial release.  By introducing a new process that combines both of these, it is believed we will trade some initial complexity for an approach that can keep up with the complexities of dealing with committee data.

## Drawbacks

- This approach is more complicated than it might otherwise be if we felt that relying entirely on scraped data (a la bills) was viable for committees.  As a result it will require some additional upfront infrastructure work to achieve the hybrid scraped/manual data approach that is desired.
- There will be ongoing maintenance costs to the new infrastructure and scrapers required for this.
- As noted above plan does not account for historical committee information.

**Backwards Compatibility Note:** The new committee scraping code will not be backwards compatible with the old.  The API and website will be updated in a way that doesn't disturb any public interfaces.

## Implementation Plan

James will lead infrastructure implementation.

Several team members, and hopefully some community members, will help to contribute updated committee scrapers.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
