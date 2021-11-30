# OSEP #1: Purpose & Process

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Final    |
| **Draft PR(s)**    |   n/a      |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/1 |
| **Created**        | 2021-02-11 |
| **Updated**        | 2021-08-17 |

---

## Abstract

Provide a formal mechanism by which major changes to Open States can be agreed upon & made.

## Specification

### Format

A proposal should have this structure:

1. **Short title** (e.g. "Adding Effective Dates"), which is also in the filename (e.g. `007-adding-effective-dates.md`).
2. **Metadata**: listed at the top of the file, to include:
	- **Author(s), Implementer(s)**  - these can be multiple project members, implementer can be blank initially.
	- **Created, Updated**: Updated should be updated when substantial changes are adopted.
	- **Status**: Draft, Accepted, Rejected, Withdrawn, Final, Superseded
	- **Issue**: To better track the current status of various proposals, all proposals must have a linked issue with the appropriate status tag.
	- **Draft PR**: Link to Draft PR discussion.
	- **Approval PR**: Link to Approval PR discussion.
3. **Abstract** - Concise summary of the proposal.
4. **Specification** - Detailed description of how the proposal should work.
5. **Rationale** - Explanation of why this would be a good idea.  The rationale should discuss considered alternatives and why they were rejected.
6. **Drawbacks** - A discussion of backwards compatibility, maintenance cost, etc.  *(Can be omitted or very simple in drafts.)*
7. **Implementation Plan** - An estimate of what it would take to implement this.  *(Can be omitted or very simple in drafts.)*
8. **Copyright** - Each proposal must end with a [CC0](https://creativecommons.org/publicdomain/zero/1.0/deed) dedication.

### Issue Status


### Process

**Drafting A Proposal**

Submitting a proposal will take a little bit more planning than just opening a GitHub issue,
but you can start there by opening a "pre-draft" issue if desired.

Issues on this repository can be used as a form of proto-proposal in the early planning phase.
No specific format is required, but these can be useful to gauge interest in a formal proposal if desired.

Small non-breaking changes and minor site & API enhancements will continue to be filed as typical issues in
[openstates/issues](https://github.com/openstates/issues/issues) but larger feature requests and similar will be moved to this repository with a note explaining this process.

When ready, draft proposals begin as PRs against this repository.
Once a proposal is deemed on-topic and in the above format, it will be accepted as a draft proposal.

**Approval Phase**

Once accepted as a draft, a full discussion will ensue.
This repository should serve as the system of record for this discussion.
(i.e. If a conversation requires a Slack conversation or similar, the results of that conversation should be attached to the relevant issue/PR)

If fields like implementer, drawbacks, and implementation plan were not specified in the draft, they must be determined before a proposal can be accepted.

After any requisite back & forth, the active core team will submit their decision.
All active core developers can review, with a goal of consensus, but in cases of indecision the project lead will make the final decision.
The status will then be adjusted to Accepted, Rejected, or Withdrawn.

**Implementation Phase**

Once accepted, work can begin as needed.
The proposal can & will likely be edited during this time.
Once the work is reviewed and accepted, the proposal's status will be updated to Final.

## Rationale

Both over the years, and right now in GitHub issues & discussions there are a lot of proposals to improve the project in various ways.

These are typically handled via a combination of GitHub issues, Slack conversations, and individual whims. By introducing a formal but lightweight Open States Enhancement Proposal (OSEP) process, I am hoping to achieve three outcomes:

1. Structure the conversation around improvements so that new contributors and core members alike can understand the scope of a proposed change.
2. Provide a means by which consensus can be gathered publicly, in a timely manner to reduce stalled decisions so that fewer issues are left in the "yeah.. we really should figure out how to X" state.  Available development time will still be an obstacle at times, but at least when time is available it will be clear what can be worked on.
3. Keep a record of failed proposals so we retain institutional memory why some things aren't done.  This can reduce the same conversation happening when new contributors arrive, and also let us revisit old decisions to see if the context around old decisions still holds or if there's need to re-evaluate.

### History

There used to be a process surrounding OCDEPs by which the core schema/etc. could be modified,
but that process was cumbersome and slow and wasn't reflective of the needs of Open States as the spec derived there was aiming at a common denominator between various team's needs.

Most other improvements have come after either lengthy conversation or on a whim, typically James'.
(Both of these come with their share of issues, hence the proposal.)

Alternatives could be to continue with the status quo or just using GitHub issues ad hoc.
Both feel inferior to having a documented process around this.

In August 2021, this proposal was updated to spell out the use of GitHub issues to track the current status of proposals.

## Drawbacks

n/a

## Implementation Plan

- If you're reading this, the repository is already created.
- Documentation and openstates/issues repo will need to be updated to point people to this process.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
