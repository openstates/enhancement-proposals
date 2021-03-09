# OSEP #2: Legal Citations

|                    |            |
|--------------------|------------|
| **Author(s)**      | Tim Showers |
| **Implementer(s)** | Tim Showers |
| **Status**         |   Draft    |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/9/ |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2020-02-17 |
| **Updated**        | TODO | 

---

## Abstract

A number of jurisdictions offer metadata to link bills to the sections of the legal code that they will alter, or to the chaptered laws. We should modify the bills model to allow scraping this data in a structured way.

**Note**: There are many different legal structures that can be altered by bills. In this document, "legal code" will be used as shorthand for "collection of laws or rules modified by a bill" such as statutes, codes, slip laws, public laws, register entries, constitutions, etc.

## Specification

Add an underlying data structure for one or more legal citations to the bill model.

These will be accessed via a new method on the bill object:

1. add_citation()

### Legal Citation:

#### Structure: 

A ```list``` of 0+ Citation ```dict```s

- **publication** - string - The affected publication. e.g. "Minnesota Statutes", "California Public Utilities Code", "DC Register", "Constitution of Nevada". Note that these cover a wide variety of different types of law and rule making.
- **citation** - string - The reference to the (sub)section of the publication. Formats and abbreviations vary widely.
- **type** - enum [`proposed`, `chapter`, `final`, `other`] - whether the citation is a part of a pending bill, a chapter law, or a final affected section of some publication after the bill as been made into law. The list of proposed citations may not match the final list due to changes between bill versions. See the `Chapter Citations` section of rationale below for further discussion of chapter laws. `other` would cover corner cases such as a constitutional amendment that was passed by the legislature, but required a referendum as well.
- **effective** - **optional** datetimeoptional - effective date
- **expiration** - **optional** datetimeoptional - expiration date
- **url** - **optional** string - Link to the URL of the affected code

#### Examples

[2019 WY HB 4](https://wyoleg.gov/Legislation/2019/HB0004)
Chaptered to CH0024 of 2019, Effective 7/1/2019

```python
bill.add_citation(
    "Wyoming Chapter Laws of 2019",
    "CH0024",
    type="chapter",
    effective=datetime(2019,07,01)
)
```


[2020 MN HF 4285](https://www.revisor.mn.gov/bills/bill.php?b=house&f=HF4285&ssn=0&y=2020)
Chaptered to "Chapter 89", modified a number of Statutes, effective 08/01/20

```python
bill.add_citation(
    "Minnesota Session Laws, 2020",
    "Chapter 89",
    type="chapter",
    effective=datetime(2020, 08, 01),
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/",
)

bill.add_citation(
    "Minnesota Statutes 2018",
    "Chapter 27, Section 27",
    type="final",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)

bill.add_citation(
    "Minnesota Statutes 2018",
    "Chapter 27, Section 13",
    type="final",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)

bill.add_citation(
    "Minnesota Statutes 2018",
    "Chapter 27, Section 17",
    type="final",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)
```

[23rd Council DC 997](https://lims.dccouncil.us/Legislation/B23-0997) which results in a DC register entry in vol 67.

```python
bill.add_citation(
    "DC Register",
    "Vol 67 and Page 14429",
    type="final",
    expires=datetime(2021,03,06)
)
```

[MO 2020 HJR 104](https://house.mo.gov/Bill.aspx?bill=HJR104&year=2020&code=R) a (failed) constitutional amendment.

```python
bill.add_citation(
    "Constitution of Missouri",
    "Article X Section 6",
    type="proposed",
)
```

## Rationale

We want to be able to catch this both proactively -- "Tell me if a bill is introduced that would alter the California corporations code", and historically -- "Who sponsored the bill that made (X) a misdemeanor instead of a felony?".

This also allows us to provide interesting session overview data -- "What laws were changed as a result of the 2019 session?", "What legislator introduced the most changes?".


### Chapter Citations:

Many jurisdictions keep a running list of all the bills that have become law in a session,
often with effective dates and redlines to aid legal researchers. These are generally known as 'chaptered laws', and may or may not also link to the state codes. Sometimes these are a holding area for laws to folded into state code(s) at a later date for something like an annual printing, but not always.

Some states (MN) chapter things like budgets that aren't statutory, so there's a chapter law without a corresponding change to a legal code.

Note that the "Chaptered Laws" of a given session, and a legal code that might have structural elements called "chapters" aren't necessarily related. So 2019/CH24 may not modify chapter 24 of some title of a legal code.

[Example chapter list from MN](https://www.revisor.mn.gov/laws/2020/0/)

### Granularity of data

A given citation may be provided at multiple levels of granularity; for example:

```
"Minnesota Statutes 2018, section 31"
```

vs 

```
"Minnesota Statutes 2018, section 31A.10",
"Minnesota Statutes 2018, section 31A.14",
"Minnesota Statutes 2018, section 31B.10-12"
```

There's often a comparatively simple first-pass approach that will yield the first result, while the second result requires a full-featured legal citation parser. The second approach also generally requires downloading and processing bill texts, which we prefer to keep seperate from the main bill scrapers for operational reasons. 

Here we leave the granularity up the individual scraper and contributor, allowing for maximum flexibility at the cost of some uniformity. We want to collect the 'low hanging fruit' of references that are on pages we already routinely scrape or could easily collect without exploding the volume of scraper requests.

If a future contributor wants to add a more fully featured legal parser to one or more scrapers, we would need to balance code complexity and scrape time against the value of the collected data. This could potentially be integrated via a new module, or a processing pipeline step that lived seperately from the existing scrapers.

Further discussion of this issue can be found the initial draft's [pull request](https://github.com/openstates/enhancement-proposals/pull/9).

### Formatting as open-text

In line with the differing formats provided by jurisdictions, consumers have differing opinions on the most granular citations that are appropriate, and how they should be formatted.

A structured approach was considered, for example:

```
{'chapter': '12', 'section': '23A', 'subsection': '3', 'paragraph': '1'}
``` 

One consumer maybe interested in the specific paragraph for redline purposes, while another may want to point to the subsection or even section for providing context around the change. 

The large number of possible formats, and inconsistent naming ('title' lives at different levels across jursidictions, for example) makes writing parsers and validators for this impractical, particularly if end consumers are likely to re-parse this data anyway. 

Requiring 'parsed' citations also significantly raises the barrier to entry to adding this data to a scraper, and likely increases scraper failures as new citation formats are encountered.

Given these drawbacks, an unstructured string is a more effective format here.

See the Formatting section in drawbacks for further details.


## Drawbacks

1. Data Availability -- The availability of this data is limited. See Note[1] below for a non-exhaustive survey.
2. Formatting - State legal citation formats are extremely varied. See Note[2] for some examples. Rather than try to standardize them, I propose we just take the states citations as is and leave followup steps to the consumer.
3. Codes/Laws are confusing. States handle rulemaking in a variety of formats -- Constitutions, Statutes, Codes, Administrative Registers, Administrative Rulemaking, etc. 
Many states also split their lawmaking into multiple publications, e.g. "criminal code", "corporations code", "environmental code", etc. Rather than try to handle that extreme complexity for a few jurisdictions, plaintext citations have us punt it to the end consumer.
4. Proposed legal citations don't seem to be available as structured data in any states right now so this would require regexing bill text which has historically not been a thing we've done in scrapers. I think it's still better to have the ability in the model.

Backwards Compatibility Issues: None, except for one-time a mass 'updated at' change in the affected states as the scrapers are updated.

## Implementation Plan

Tim adds the model to openstates-core, with some help from James on questions.

James updates the poetry requirements and rolls new images, and does database migrations.

Scraper writers add the functionality to states as desired. Tim would take the first shot at the known states.

## Notes

1. States that offer chapters post hoc -- DC, GA, US, WY, MN
2. Projects that try to parse legal citations: [law-identifier](https://github.com/statedecoded/law-identifier), [citation-regexes](https://github.com/freelawproject/citation-regexes)

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
