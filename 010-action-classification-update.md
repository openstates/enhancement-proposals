# OSEP #1: Purpose & Process

|                    |            |
|--------------------|------------|
| **Author(s)**      | @sruthivedantham |
| **Implementer(s)** | @sruthivedantham |
| **Status**         |      |
| **Draft PR(s)**    |  [#4152]([url](https://github.com/openstates/openstates-scrapers/pull/4152)), [#4154 ]([url](https://github.com/openstates/openstates-scrapers/pull/4154)), [#4155]([url](https://github.com/openstates/openstates-scrapers/pull/4155))      |
| **Approval PR(s)** |  |
| **Created**        | 2022-03-11 |
| **Updated**        | 2022-03-11 |

---

## Abstract

Action classification for bills largely happens inside of the scrape() method. I propose we isolate everything related to action classification in its own file for each jurisdiction. 

## Specification
In this new design, all code related to action classification will be moved to a separate file. This will help untagle action classification from scraping, as well as make the scrape() methods simpler and easier to read.
The contents of the action classification file should be standardized across jurisdictions to contain: 
- a dictionary that matches action phrases from the bill to defined [OS classifications]([url](https://github.com/openstates/openstates-core/blob/5b16776b1882da925e8e8d5c0a07160a7d649c69/openstates/data/common.py#L87))
- a function that takes in a phrase and returns the appropriate classification
- anything else that is required for action classification

For example, the mapping dictionary in the actions.py for Alaska would look like this:
```
_actions = {
    "bill action phrase": {"type": "string comparison or regex", "mappings": ["OS classification mapping"]},
    "read the second time": {"type": "compare", "mappings": ["reading-2"]},
    "^introduced": {"type": "regex", "mappings": ["introduction"]},
    }
```

## Rationale
Action classification is hard because there are so many things that vary from jurisdiction to jurisdiction. By implementing these changes, we can help standardize the process, even if the content remains variable. 
Additionally, moving all action classification code for each jurisdiction to its own actions.py will help simply the scrape() method, which can be very long and complex for some jurisdictions. 


## Drawbacks

- Every scraper is different and action classification is done differently from one to another. Pulling action classification code out of the `scrape()` method is easier for some jurisdictions than others.

## Implementation Plan

- This would ideally need to be implemented in every jurisdiction's scraper (although some already achieve this to a greater degree than others). 

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
