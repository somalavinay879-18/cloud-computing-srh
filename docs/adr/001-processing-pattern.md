# ADR 001: Lambda architecture with a lightweight speed layer

## Status
Proposed

Date: 2026-11-20

## Context
Launch customer: **SHP02 Nordic Home**, a furniture chain that plans new store openings
around container arrivals. Ocean only, 4 lanes. Targets:

- **Freshness:** 1 hour
- **Reliability:** 75% on time on every lane
- **Budget:** $45 per month

Delay per lane, October 2026 (`docs/metrics.md`):

| Lane | Avg delay (h) | On time | vs 75% |
|---|---|---|---|
| KRPUS → USLGB | 55.5 | 45.7% | misses |
| NLRTM → USHOU | 42.3 | 63.2% | misses |
| DEHAM → USNYC | 53.8 | 63.3% | misses |
| SGSIN → BEANR | 6.6 | 100.0% | meets |

Three of four lanes miss the target, mostly because of carriers HRZ, KST, NWL and ORS on
specific lanes. So the platform has two jobs: warn operations within an hour when a container
is running late (live), and measure carrier reliability per lane over weeks (batch). The two
kinds of data also arrive differently: shipment events one at a time, and schedules, bookings
and rates as daily files.

## Options considered
1. **Lambda:** a speed layer for shipment events and a separate batch layer for the daily
   files, both writing to BigQuery. Matches how the data arrives, but means two paths to build
   and keep consistent.
2. **Kappa:** everything is a stream, including the daily files replayed as events; one
   streaming pipeline (for example Dataflow) computes both the live view and the reliability
   metrics, and history is recomputed by replaying the log. One code path, but it needs an
   always-on stream processor and turns simple daily files into a stream.
3. **Batch only:** load everything hourly. Cheapest, but an hourly load just meets the 1-hour
   target with no margin, and events would wait up to an hour before anyone sees them.

## Decision
We chose **Lambda (option 1), with a lightweight speed layer**: Pub/Sub writes shipment events
straight into BigQuery through a BigQuery subscription, with no Dataflow or Kafka. The batch
layer loads the daily files into BigQuery and builds the reliability tables with scheduled SQL.

Reasons:
- The data arrives in two shapes, so two paths match the sources; Kappa would force daily files into a stream for no gain.
- Nordic Home's freshness target is 1 hour, not seconds, so the speed layer does not need a stream processor.
- The reliability metrics are computed over weeks, which is a batch job by nature.
- Without always-on streaming workers, the platform can stay under $45 per month.

## Consequences
- **Cost:** Pub/Sub, BigQuery storage and queries, and Cloud Storage for a few MB a month should
  stay well under $45. An always-on Dataflow streaming job would likely use most of the budget
  by itself; check both in the Google Cloud pricing calculator.
- **Freshness:** events reach BigQuery within seconds, well inside the 1-hour target; the
  reliability scorecard is up to a day old, which is fine for a weekly carrier decision.
- **Complexity:** two paths to monitor, and the live table and the curated tables must use the
  same definition of "on time" (arrival no more than 24 hours late).
- **What gets harder:** transforming events in flight (enrichment, deduplication, windowed
  alerts) has to happen in SQL after landing, or we add Dataflow later in a new ADR. If Nordic
  Home tightens freshness to minutes with complex rules, this decision should be revisited.
