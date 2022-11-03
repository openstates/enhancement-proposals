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

Action classification for bills largely happens inside of the `scrape()` method (or somewhere within `bills.py`). I propose we isolate everything related to action classification in its own file for each jurisdiction. 

## Specification
In this new design, all code related to action classification will be moved to a separate file (`actions.py`). This will help untangle action classification from scraping, as well as make the `scrape()`/`bills.py` methods simpler and easier to read.
The contents of the action classification file should be standardized across jurisdictions to contain: 
- a dictionary that matches action phrases from the bill to defined [OS classifications]([url](https://github.com/openstates/openstates-core/blob/5b16776b1882da925e8e8d5c0a07160a7d649c69/openstates/data/common.py#L87)) with the action phrase as keys (ie "read for the first time"). The value of the keys would be a dictionary that contains:
     - “Type” (how to match action classification phrase, currently either regex or string comparison). Ex: “Regex” or “Compare”
     - “Mappings” (respective mapping from OS classifications). Ex: [“reading-1”, “introduction”]

- a function that takes in a phrase and returns the appropriate classification
- anything else that is required for action classification

While classifying the action (in the `categorize_action()` function), we use the `type` dictionary value to determine whether we need to use regex or a simple string comparison to determine whether an identifying bill action phrase can be found within the bill action statement. 

For example, the mapping dictionary in the `actions.py` for Alaska would look like this:
``` python
_actions = {
    "bill action phrase": {
        "type": "string comparison or regex", 
        "mappings": ["OS classification mapping"]
        },
    "read the second time": {
        "type": "compare", 
        "mappings": ["reading-2"]
        },
    "^introduced": {
        "type": "regex", 
        "mappings": ["introduction"]
        },
    }
```

An example action classification function in a jurisdiction's `actions.py` would look like this: 
``` python
def categorize_actions(action_description):
    atype = []
    for action_key, data in _actions.items():
        # If regex is required to isolate bill action phrase
        if data["type"] == "regex":
            if re.search(action_key, action_description.lower()):
                atype.extend(a for a in data["mappings"])

        # Otherwise, we use basic string comparison
        else:
            # If we can detect a phrase that there is an OS action classification for
            if action_key in action_description.lower():
                atype.extend(a for a in data["mappings"])

    return atype
```

## Rationale
Action classification is hard because there are so many things that vary from jurisdiction to jurisdiction. By implementing these changes, we can help standardize the process, even if the content remains variable. 

Additionally, moving all action classification code for each jurisdiction to its own `actions.py` will help simply the `scrape()` method/`bills.py`, which can be very long and complex for some jurisdictions. We are currently working to [enable post-processing per jurisdiction]([url](https://github.com/openstates/enhancement-proposals/blob/93c7e97da8378bbeea200bdc857a536a63d0b465/010-reprocessing.md)), which means that post-processors will run as the scrape runs. Whereas right now the scrape writes objects directly to disk, the scrape step will be updated to instead send the ScrapedBill through a chain of configured post-processors (in this case Actions per Jurisdiction). Moving action classification code to its own `actions.py` will assist in this. 


## Drawbacks

- Every scraper is different and action classification is done differently from one to another. Pulling action classification code out of the `scrape()` method is easier for some jurisdictions than others.

## Implementation Plan

- This would ideally need to be implemented in every jurisdiction's scraper (although some already achieve this to a greater degree than others). 

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
