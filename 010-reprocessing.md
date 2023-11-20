# OSEP #10: Data Reprocessing

|                    |            |
|--------------------|------------|
| **Author(s)**      | @jamesturk |
| **Implementer(s)** | @jamesturk |
| **Status**         |   Draft    |
| **Issue**          | https://github.com/openstates/enhancement-proposals/issues/TBD |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2021-12-03 |
| **Updated**        | 2021-12-03 |

---

## Abstract

This is a proposal to define an interface for data transformations that run against scraped data.  By defining this transformation step as a distinct part of our pipeline, it will be possible to reprocess bills without re-running the scrapers.

## Specification

Right now there are a few types of data classification & augmentation that would be better handled as a distinct step:

- action classification (part of the scrape)
- sponsor & voter matching (part of the import)
- action linkage to committees (part of the import)
- possibly effective date handling (https://github.com/openstates/enhancement-proposals/issues/11)
- in the past there were others as well, and there could be more in the future, for example:
	- subject normalization used to work very similarly to action classification, mapping all subjects to a curated list
	- other `classification` fields might be suited to this approach

This proposal suggests adding a step after scrape that would apply these transformations.

### Post-processor Interface
One component of this proposal will be to add a post-processor interface that runs on a per-object basis for bills and votes.

Essentially an interface that looks something like:
```
def bill_postprocessor(bill: ScrapedBill) -> ScrapedBill
```

where `ScrapedBill` is a representation of the bill as scraped, possibly already transformed by other processors.

### Data Flow Changes

Right now when an object is yielded back from a scraper it goes through the following steps:

(save_object @ openstates/scrape/base.py#117)

1. Whitespace is cleaned.
2. The type-specific pre_save function is called, which sanitizes the object.
3. A JSON filename is determined.
4. Debugging output is printed.
5. The raw scraped output is emitted:
	a. If there is a configured `SCRAPE_OUTPUT_HANDLER` environment variable, its `handle` method is called.
	b. Otherwise, it is written to disk as JSON.
6. The `validate` method is called.  (This occurs after writing to disk so that invalid output can still be examined.)
	a. if validation fails, processing will exit here (unless strict_validation is disabled, which may not be a thing in practice?)
7. finally, subordinate objects are saved (this is when votes attached to a bill are saved by the same set of steps)

This proposal would alter this flow to:

1. Optionally emit non-cleaned JSON (controlled by a flag or environment variable).
2. The validate method is called.  (This is moved forward so that the postprocessing functions can know they are dealing with a valid object.)
3. The configured post-processing chain runs on the object in question:
	(Whitespace cleaning and other pre_save sanitization is moved to an `openstates.postprocessing.default_bill_postprocessor` function or equivalent.)
4. The validate method is called again, ensuring that the object remains valid throughout the transformations.
5. `SCRAPE_OUTPUT_HANDLER` has its `handle` method called on the final object resulting from the transformation chain.  The current separate branch (6b above) is instead rewritten to perform as one of several possible output handlers.
6. Finally, subordinate objects will be saved as they are now.

This means that post-processors will run as the scrape runs.  Whereas right now the scrape writes objects directly to disk, the scrape step will be updated to instead send the `ScrapedBill` (or `ScrapedVote`) through a chain of configured post-processors.  The result of that output will be written to disk.

This allows for the import process to not be touched by this change, as it can continue to assume that the JSON data (or equivalent) is the data it is meant to import.

Post-processors may need to be enabled/disabled on a system-wide basis, or per-jurisdiction.  These needs will be met by adding both an additional top level setting (`openstates.settings.POSTPROCESSORS`/`os.environ['OPENSTATES_POSTPROCESSORS']`) which will be a list of post-processing functions in dotted form (e.g. `"openstates.postprocessors.SponsorLinker", "nh.NewHampshireActionClassifier"`).

Should a particular Jurisdiction need to add to this list, the `Jurisdiction` class will gain a property `additional_postprocessors` which will contain its own list of post-processing functions which will be called after all global post-processors.

For debugging purposes, it will be possible to disable the post-processor chain by running with `--no-postprocess`.

TODO: This is the simplest implementation I think, but do we need a way to disable global post-processors for given jurisdictions?  Do we need the ability to change the order and insert additional post-processors within the chain?  If so we'll probably want to remove the global post-processors and favor per-jurisdiction configuration.

### Running post-processing functions against scraped data

During development, there is a desire to be able to re-run processing functions against scraped data.

This can be achieved by adding a command line flag to the `os-update` CLI.  Usage might look something like:

```
$ os-update usa bills --scrape --no-postprocess
... (data emitted to _data/usa/) ...

$ os-update usa bills --postprocessor my.development.postprocessor
... (data in _data/usa/ updated) ...
```

Again, this is intended only for development purposes, once a post-processor is ready, it should be added to the global or jurisdiction's configuration.

### Running post-processing functions against database

Ultimately the main goal of this refactor is to make it possible to re-run post-processor functions against already scraped data without the original JSON.

The main challenge with this is that the data is somewhat transformed by the time it is in the database.  In order for the post-processor functions to not need to know whether or not the data is coming from scraped JSON or the database (or potentially other sources depending on the `SCRAPE_OUTPUT_HANDLER`) this feature will require adding functionality that will convert a database `Bill` back to a `ScrapedBill`.

Once that is in place, it should be possible to run a reprocess command specifying a subset of data to operate on and post-processor to re-run.  That would then perform the following:

1.  A database query is done that selects the data to be modified, prompting the user to verify that it looks correct.
2.  Each selected object is run through a `to_json` method that converts the data back to the form it was in when scraped.  (In practice there will likely be small differences, but the field names, etc. should be identical so that the function need not know the source of its data.)
3.  The resultant `ScrapedBill` object will then be passed through the post-processors.
4.  The `ScrapedBill` objects are then re-imported.

TODO: Another option here would be to store the pre-transformed data as well.  If we did that, it'd be stored as a JSON blob.  That'd be less error-prone at the cost of some additional storage.  Open to discussion.

## Rationale

The current ad-hoc approach presents a few issues:
- Scrapers can not always be re-run, especially not in a timely fashion.  But sometimes data is taken down from source sites entirely.
- Re-running importers requires keeping around intermediate data.
- Doing development work on either requires repeatedly running a scrape.
- Scraper code is littered with classification conditionals & maps, and repeated (& error prone) logic that could be centralized and tested.

Additionally, the desire is to be able to run these processes separately from the scrape or import steps to re-process data.

## Drawbacks

TBD

The main drawback that comes to mind is just the increased complexity, suggestions & ideas on where we might make simplifications are very welcome.

## Implementation Plan

James will take the lead on the os-core changes.  Once this functionality is available, scrapers will have their action classifiers ported as needed.

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
