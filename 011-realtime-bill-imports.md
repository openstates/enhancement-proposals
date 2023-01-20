# OSEP #11: Openstates Realtime Bill Imports

|                    |                                                                |
|--------------------|----------------------------------------------------------------|
| **Author(s)**      | @elseagle                                                      |
| **Implementer(s)** | @elseagle                                                      |
| **Status**         | Done                                                           |
| **Draft PR(s)**    | [#5](https://github.com/openstates/openstates-realtime/pull/5) |
| **Approval PR(s)** | [#5](https://github.com/openstates/openstates-realtime/pull/5) |
| **Created**        | 2023-01-06                                                     |
| **Updated**        | 2023-01-20                                                     |

______________________________________________________________________

## Abstract

We are proposing a realtime ingestion feature for openstates-scraper outputs.

## Specification

We want to reduce the time it takes for openstates-scraper outputs to be imported. This will allow us to ingest new
bills, events, vote_events, and people as they are scraped, rather than waiting for the import to run at the end of
the scrape cycle. This will allow us to provide more timely data to users. This [one-pager](https://civic-eagle.atlassian.net/wiki/spaces/PD/pages/1431928835/Openstates+Scraper+Realtime+Ingestion+Feature)
provides more context on the motivation for this feature.

## Implementation Plan

This feature will be implemented in three parts:

- Openstates-Core
- Openstates-Scraper
- Openstates-Realtime (a new lambda service)

### Openstates-Core

Openstates-Core will be updated to support realtime ingestion. This will include:

- Adding a new `--realtime` flag to the arguments used in the run command. This flag will be used to determine
  whether to
  run the realtime ingestion process or the normal ingestion process.
- The flag will ensure that files are uploaded to s3 and s3 file paths are sent to SQS.
- It will also ensure that no imports are done after the scrape currently being run is complete.

### Openstates-Scraper

On openstates-scraper will be setting the environment variables needed to run the realtime ingestion process. The
environment variables are:

- S3_REALTIME_BASE: The path to the s3 bucket e.g s3://openstates-realtime-bills-dev
- AWS_ACCESS_KEY_ID: The AWS access key id
- AWS_SECRET_ACCESS_KEY: The AWS secret access key
- AWS_DEFAULT_REGION: The AWS region
- SQS_QUEUE_URL: The SQS queue url

### Openstates-Realtime

Openstates-Realtime will be a new lambda service that will be responsible for importing the realtime data. It will
be based off SQS and will be triggered by a cron job. The cron job will be set to run every 2 minutes. The lambda
will load the data from s3 and import it into the database. The lambda will also be responsible for deleting the
messages from the queue once they have been processed. A maximum of 300 messages will be processed at a time to
avoid multiple calls to the database concurrently. Up to 10 instances of this lambda server can be spun up in a
scenario where the import isn't complete within 2minutes, which implies we can import 3000 bills in concurrently
with just 10 connections to the db. This lambda also moves files from the root s3 bucket folder for the jurisdiction
being run to the `archived` folder. This is to ensure that the files are not processed again if
archived.

## Rationale

This will help us reduce the time it takes for openstates-scraper outputs to be imported. This will allow us to have
reduction in missing bills time metric.

## Drawbacks

- This feature will require a lot of changes to the openstates-scraper and openstates-core codebases. This will
  require a lot of testing to ensure that the realtime ingestion process does not break the normal ingestion
- The cost of running this feature will be higher than the cost of running the normal ingestion process. This is
  because the realtime ingestion process will be running more frequently. This will also require more resources
  to be allocated to the realtime ingestion process.

## Alternatives

- S3 bucket as a temporary storage that each output is dumped into while the scrape runs progress. A (serverless)
  lambda function automatically listens to “events” on this bucket, picks up newly dumped files and ingests them into
  the Openstates DB in real-time.
- A background job that processes each output from a cache/queue which the scraper will dump to as it runs. This job
  will automatically ingest all outputs dumped and clear the queue.

## Unresolved Questions

- How do we measure the performance of this feature in terms of the improvement it brings to the time it takes for
  data to be available to users?
- Would we need to move this away from AWS in the future ?
- Is there a better name we could give the flag other than `--realtime` that could make it more descriptive ?

## Copyright

This document has been placed in the public domain per the [Creative Commons CC0 1.0 Universal license.](https://creativecommons.org/publicdomain/zero/1.0/deed)
