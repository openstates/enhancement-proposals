# OSEP #10: Action Classification Update

|                    |            |
|--------------------|------------|
| **Author(s)**      | @sruthivedantham |
| **Implementer(s)** | @sruthivedantham |
| **Status**         |      |
| **Draft PR(s)**    |  ([#4152](https://github.com/openstates/openstates-scrapers/pull/4152)), ([#4154](https://github.com/openstates/openstates-scrapers/pull/4154)), ([#4155](https://github.com/openstates/openstates-scrapers/pull/4155))      |
| **Approval PR(s)** |  |
| **Created**        | 2022-03-11 |
| **Updated**        | 2022-25-11 |

---

## Abstract

Action classification for bills largely happens inside of the `scrape()` method (or somewhere within `bills.py`). I propose we isolate everything related to action classification in its own file for each jurisdiction. 

## Specification
In this new design, all code related to action classification will be moved to a separate file (`actions.py`). This will help untangle action classification from scraping, as well as make the `scrape()`/`bills.py` methods simpler and easier to read.

The contents of the action classification file should be standardized across jurisdictions to utilize the [BaseCategorizer](https://github.com/openstates/openstates-scrapers/blob/070b9d4a77f835ecf369feb4399ebc71fac20bc1/scrapers/utils/actions.py#L62) and customize rules as appropriate. Regex patterns will match action phrases to defined [OS classifications](https://github.com/openstates/openstates-core/blob/5b16776b1882da925e8e8d5c0a07160a7d649c69/openstates/data/common.py#L87). 

Every actions.py will contain:
- custom [rules](https://github.com/openstates/openstates-scrapers/blob/070b9d4a77f835ecf369feb4399ebc71fac20bc1/scrapers/utils/actions.py#L6) that apply to the jurisdiction's bill language
- a `categorize_action()` function that takes in a phrase and returns the appropriate classification utilizing the BaseCategorizer class
- anything else that is required for action classification


Examples of Rules that could be found in an actions.py include:
``` python
    Rule(r"amendment not adopted", "amendment-failure"),
    Rule(r"(?i)third reading, (?P<pass_fail>(passed|failed))", "reading-3"),
    Rule(r"Read first time", "reading-1"),
    Rule(r"(?i)first reading, referred to (?P<committees>.*)\.", "reading-1"),
    Rule(r"(?i)And refer to (?P<committees>.*)", "referral-committee"),
```

An example action classification function in a jurisdiction's `actions.py` would look like this: 
``` python
    def categorize(self, text):
        """Wrap categorize and add boilerplate committees."""
        attrs = BaseCategorizer.categorize(self, text)
        if "committees" in attrs:
            committees = attrs["committees"]
            for committee in re.findall(committees_rgx, text, re.I):
                if committee not in committees:
                    committees.append(committee)
        return attrs
```

## Rationale
Action classification is hard because there are so many things that vary from jurisdiction to jurisdiction. By implementing these changes, we can help standardize the process, even if the content remains variable. 

Additionally, moving all action classification code for each jurisdiction to its own `actions.py` will help simply the `scrape()` method/`bills.py`, which can be very long and complex for some jurisdictions. We are currently working to ([enable post-processing per jurisdiction](https://github.com/openstates/enhancement-proposals/blob/93c7e97da8378bbeea200bdc857a536a63d0b465/010-reprocessing.md)), which means that post-processors will run as the scrape runs. Whereas right now the scrape writes objects directly to disk, the scrape step will be updated to instead send the ScrapedBill through a chain of configured post-processors (one of which would be Actions per Jurisdiction). Moving action classification code to its own `actions.py` will assist in being able to eventually link it in as an individual post-processor. 


## Drawbacks

- Every scraper is different and action classification is done differently from one to another. Pulling action classification code out of the `scrape()` method is easier for some jurisdictions than others.

## Implementation Plan

- This would ideally need to be implemented in every jurisdiction's scraper (although some already achieve this to a greater degree than others). 

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
