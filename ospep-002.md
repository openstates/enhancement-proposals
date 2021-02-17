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

A number of jurisidictions offer metadata to link bills to the sections of the legal code that they will alter, or to the finished chaptered laws. We should modify the bills model to allow scraping this data.


## Specification

We'd add 2 new underlying data structures, a chapter cite and a legal cite:

###Chapter Citation:
```
str - chapter 
str - session
optional datetimeoptional|str - effective
optional datetimeoptional|str - expires
optional url
```

Example:

[2019 WY HB 4](https://wyoleg.gov/Legislation/2019/HB0004)
Chaptered to CH0024, Effective 7/1/2019

```
bill.add_chapter(
    name="CH0024",
    session="2019",
    effective=datetime(2019,07,01)
)
```

### Legal Citation
```
str - code_name
str - citation
optional datetimeoptional|str - effective
optional datetimeoptional|str - expires
optional url
```


[2020 MN HF 4285](https://www.revisor.mn.gov/bills/bill.php?b=house&f=HF4285&ssn=0&y=2020)
Cited to "Chapter 89, Article 4, Section 34", effective 08/01/20

```
bill.add_legal_citation(
    name="Laws of Minnesota",
    citation="Chapter 89, Article 4, Section 34",
    effective=datetime(2020, 08, 01)
    url="https://www.revisor.mn.gov/laws/2020/0/Session+Law/Chapter/89/"
)
```

https://www.revisor.mn.gov/bills/bill.php?b=house&f=HF4285&ssn=0&y=2020
"Laws of Minnesota"
"Chapter 89, Article 4, Section 34"

https://lims.dccouncil.us/Legislation/B23-0997
"DC Register"
"Vol 67 and Page 14429"
exp: Mar 06, 2021


## Rationale

We want to be able to catch this both proactively -- "Tell me if a bill is introduced that would alter the California corporations code", and historically -- "Who sponsored the bill that made (X) a misdemeanor instead of a felony?"

## Drawbacks

1. Data Availability -- The availability of this data is limited. See Note[1] below for a survey.
2. Formatting - State legal citation formats are extremely varied. See Note[2] for some examples. Rather than try to standardize them, I propose we just take the states citations as is and leave followup steps to the consumer.
3. Codes/Laws are confusing. States handle rulemaking in a variety of formats -- Constitutions, Statutes, Codes, Administrative Registers, Administrative Rulemaking, etc. Rather than try to handle that extreme complexity for a few jurisdictions, plaintext citations have us punt it to the end consumer.

Backwards Compatibility Issues: None, except for one-time a mass 'updated at' change in the affected states.

## Implementation Plan

Tim adds the model to openstates-core, with some help from James on questions

## Notes

1. States that offer chapters post hoc -- DC, GA, US, WY, MN
2. Projects that try to parse legal citations == [law-identifier](https://github.com/statedecoded/law-identifier), [citation-regexes](https://github.com/freelawproject/citation-regexes)

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
