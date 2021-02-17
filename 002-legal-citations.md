# OSEP #2: Legal Citations

|                    |            |
|--------------------|------------|
| **Author(s)**      | Tim Showers |
| **Implementer(s)** | Tim Showers |
| **Status**         |   Draft    |
| **Draft PR(s)**    | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Approval PR(s)** | https://github.com/openstates/enhancement-proposals/pull/TBD |
| **Created**        | 2020-02-17 |
| **Updated**        | TODO | 

---

## Abstract

A number of jurisdictions offer metadata to link bills to the sections of the legal code that they will alter, or to the chaptered laws. We should modify the bills model to allow scraping this data in a structured way.

**Note**: There are many different legal structures that can be altered by bills. In this document, "legal code" will be used as shorthand for "collection of laws or rules modified by a bill" such as statutes, codes, slip laws, public laws, register entries, constitutions, etc.

## Specification

Add 2 new underlying data structures as part of the bill model, a chapter citation and a legal citation.

These will be accessed via two methods:

1. add_chapter()
1. add_citation()

### Chapter Citation:

Many jurisdictions keep a running list of all the bills that have become law in a session,
often with effective dates and redlines to aid legal researchers. These are generally known as 'chaptered laws', and may or may not also link to the state codes.

[Example from MN](https://www.revisor.mn.gov/laws/2020/0/)

#### Structure: 

A ```list``` of 0+ chapter ```dict```s


- **chapter** - string - The jurisdiction's chapter laws reference. 
- **session** - string - The session or year of the chapter law. This may be distinct from the bill's "session", as some states split their chapter laws up by calendar year.
- **effective** - **optional** datetimeoptional - effective date
- **expiration** - **optional** datetimeoptional - expiration date
- **url** - **optional** string - Link to the URL of the chapter law or the redlines.


Note that the "Chaptered Laws" of a given session, and a legal code that might have "chapters" aren't necessarily related. So 2019/CH24 may not modify chapter 24 of some title of the legal code.

#### Examples

[2019 WY HB 4](https://wyoleg.gov/Legislation/2019/HB0004)
Chaptered to CH0024 of 2019, Effective 7/1/2019

```python
bill.add_chapter(
    name="CH0024",
    session="2019",
    effective=datetime(2019,07,01)
)
```

### Legal Citation:

#### Structure: 

A ```list``` of 0+ Citation ```dict```s

- **publication** - string - The affected publication. e.g. "Minnesota Statutes", "California Public Utilities Code", "DC Register", "Constitution of Nevada". Note that these cover a wide variety of different types of law and rule making.
- **citation** - string - The reference to the (sub)section of the publication. Formats and abbreviations vary widely.
- **type** - enum [proposed, final] - whether the citation is a part of a pending bill, or a final affected section after the bill as been made into law. The list of proposed citations may not match the final list due to changes between bill versions.
- **effective** - **optional** datetimeoptional - effective date
- **expiration** - **optional** datetimeoptional - expiration date
- **url** - **optional** string - Link to the URL of the affected code


#### Examples

[2020 MN HF 4285](https://www.revisor.mn.gov/bills/bill.php?b=house&f=HF4285&ssn=0&y=2020)
Cited to "Chapter 89, Article 4, Section 34", effective 08/01/20

```python
bill.add_citation(
    "Laws of Minnesota",
    "Chapter 89, Article 4, Section 34",
    type="final",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)
```

If the bill modified multiple sections, we would call it multiple times. Note the actual example bill only targets one section, this is just an example:

```python
bill.add_citation(
    "Laws of Minnesota",
    "Chapter 89, Article 5, Section 50",
    type="final",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)
```

[23rd Council DC 997](https://lims.dccouncil.us/Legislation/B23-0997) which results in a DC register entry in vol 67.

```python
bill.add_legal_citation(
    "DC Register",
    "Vol 67 and Page 14429",
    type="final",
    expires=datetime(2021,03,06)
)
```

## Rationale

We want to be able to catch this both proactively -- "Tell me if a bill is introduced that would alter the California corporations code", and historically -- "Who sponsored the bill that made (X) a misdemeanor instead of a felony?".

This also allows us to provide interesting session overview data -- "What laws were changed as a result of the 2019 session?", "What legislator introduced the most changes?".

## Drawbacks

1. Data Availability -- The availability of this data is limited. See Note[1] below for a non-exhaustive survey.
2. Formatting - State legal citation formats are extremely varied. See Note[2] for some examples. Rather than try to standardize them, I propose we just take the states citations as is and leave followup steps to the consumer.
3. Codes/Laws are confusing. States handle rulemaking in a variety of formats -- Constitutions, Statutes, Codes, Administrative Registers, Administrative Rulemaking, etc. 
Many states also split their lawmaking into multiple publications, e.g. "criminal code", "corporations code", "environmental code", etc. Rather than try to handle that extreme complexity for a few jurisdictions, plaintext citations have us punt it to the end consumer.
4. Proposed legal citations don't seem to be available structured data in any states right now (TODO: are we sure?) so this would require regexing bill text which has historically not been a thing we've done in scrapers. I think it's still better to have the ability in the model.

Backwards Compatibility Issues: None, except for one-time a mass 'updated at' change in the affected states as the scrapers are updated.

## Implementation Plan

Tim adds the model to openstates-core, with some help from James on questions.

James updates the poetry requirements and rolls new images, and does database migrations.

Scraper writers add the functionality to states as desired. Tim would take the first shot at the known states.

## Notes

1. States that offer chapters post hoc -- DC, GA, US, WY, MN
2. Projects that try to parse legal citations == [law-identifier](https://github.com/statedecoded/law-identifier), [citation-regexes](https://github.com/freelawproject/citation-regexes)

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
