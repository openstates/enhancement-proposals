# OSEP #3: Manual People Data Tools

|                    |            |
|--------------------|------------|
| **Author(s)**      | James Turk |
| **Implementer(s)** | James Turk |
| **Status**         |   Draft    |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/14 |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-02-23 |
| **Updated**        | 2021-02-23 | 

---

## Abstract

Introduce a public means by which we can improve and enhance legislator data, one that is available to less technical volunteers.

## Specification

I am proposing we build a public tool to handle common corrections to legislator data.  This tool would be used primarily by less technical volunteers that are not comfortable with the clone-edit YAML-PR workflow that currently exists.

The tool is aimed at three types of issues in particular:

- adding details such as phone numbers, social media accounts, etc. to an individual or in bulk
- retiring legislators that have left office
- adding new one-off legislators that won special elections or appointments

It's important to note that not all issues will be considered.  For example:
- an election has occurred and we need a full update
- a systemic issue has resulted in us grabbing incorrect addresses, swapping voice/fax, or similar

These will continue to be best handled by re-scraping, and will be considered out of scope for this proposal.

What I am proposing is a publicly available web app that provides the following:

### Use Case 1: Bulk Editing & Review

This will be the primary view into the new app.  It will present the user with a table like:

| name | district | chamber |                 |
| ---- | -------- | ------- | --------------- |
|      |          |         | [Retire] [Edit] | 

There will be form controls to add editable columns including District & Capitol Voice, Fax, Address, Twitter, etc.

Ideally the tool could provide a summary table for a sense of coverage for important fields as well, e.g. (23% of legislators are missing phone number).

If a user wishes to perform a bulk edit, they will add the appropriate fields using the controls, and make the appropriate edits like they would in a spreadsheet.

At the bottom of this page, there will be a form that will ask them for a short description of their changes, including a link to sources they used.

Pressing this will save a **ChangeSet** to the local database.

Each row also has a retire & edit link.

### Use Case 2: Retirement

Another option available, is to retire a given individual.  

Clicking the retire link will prompt for the necessary details (retirement date, source, death?) and clicking save will create a simple **ChangeSet** indicating the retirement.

### Use Case 3: New Legislators (and editing a full legislator)

In addition to the legislator table, there will also be individual edit pages.  If a [New Legislator] button is pressed, the user will be prompted for the initial non-editable details such as Name, District, Chamber.

Pressing [Save] will create a **ChangeSet** with just this legislator's information.

(As noted above, this is only intended to be used for 1-2 legislators that have been appointed, a full session turnover should be processed by scraping.)

### Saving ChangeSets

Each of the above actions saves data representing its delta to the existing legislator data.  These will be stored in a local database, and converted to a PR against the openstates/people repository.

The PR will:
- provide ChangeSet represented in minimally-edited YAML
- to avoid dealing with GitHub credentials, the PR will likely be by openstates-bot or similar.
- the Open States user's username (planning to use existing OpenStates.org authentication most likely for simplicity) -- might want a feature to @ mention their GitHub account



## Rationale

After a full audit of Open States' legislator data, and reviewing the recently filed legislator issues, it seems like there are 3 types of issues that are common.  I think with a bit of work we could come up with a tool that allows less technical volunteers to handle all of these.  I think it will be worthwhile as we commit to keeping people data as up-to-date as possible.

Other options considered included:
- a locally-runnable app that worked directly on the YAML. 
	- (This likely wouldn't really reduce the barrier to entry and would still require users to have basic GitHub skills.)
- an AirTable app, as started in the [openstates/people#383 PR](https://github.com/openstates/people/pull/383). 
	- (This still requires someone running the script locally, then handing the airtable over to PRs, also there are things that are likely easier with a simple CRUD app instead of trying to bend AirTable to the schema here.)
- a web YAML editor 
	- (This doesn't really allow retirements, bulk editing, etc. so doesn't feel like as much of a win.)

## Drawbacks

There's of course added complexity in building and supporting a new tool.  I am hoping that the scope detailed here will be possible to implement a working prototype in ~2 weeks and make continued revisions as the first batch of users work with it.

This tool will not be able to do everything, so YAML will still be required for certain tasks.  While this could be seen as a drawback, this limitation is by design, since the YAML exists there is no need for the tool to make it possible to cover all the strange edge cases that exist in needing to change the data.  One example is that for now the tool will not provide a way to change a legislator's district or party as those changes are rare and a bit more complex in the YAML than others.  (It'd be possible to add those features later though if they prove necessary, but the features described above aim at 80% of the edits we want to do.)

## Implementation Plan

I'd plan to work on this myself, possibly with the assistance of one other engineer if resources were available.  If prioritized, I think a working version could be produced in the next couple of weeks, leaving room for UI and other functionality improvements down the line.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
