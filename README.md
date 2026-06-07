# YouTube Sliding Top-K Rankings

Production-oriented design notes for a YouTube-style top-k ranking system over sliding windows.

## GitHub Pages

Architecture page:

https://edisontrent17.github.io/System-Design/

Component 3 visualization:

https://edisontrent17.github.io/System-Design/component-3.html

## Core Idea

The serving API should not compute top-k rankings from raw events. It should read materialized ranking snapshots produced by a streaming pipeline.

```text
Playback Events
  -> Raw Durable Stream / Lake
  -> Candidate Compaction
  -> Validation / Deduplication / Fraud Filtering
  -> Kafka or PubSub Topics
  -> Sliding Window Aggregation
  -> Shard-Local Top-K
  -> Global Top-K Merge
  -> Ranking Snapshot Store
  -> Redis or Edge Cache
  -> Top-K API
```

## Ranking Contract

Hot rankings use bucket-aligned sliding windows. Counts are exact for processed, validated events inside full buckets up to the published watermark.

```text
rankQuality = exact_bucketed
window = [bucket_aligned_start, bucket_aligned_end)
bucketSize = explicit in every response
watermark = all accepted events with event_time <= watermark are included
```

Raw playback starts, view candidates, valid views, public counters, and correction events are intentionally different concepts.

## Critical Design Corrections

- Raw events are durably written before destructive filtering.
- Component 3 receives compacted view candidates or watch segments, not every heartbeat.
- Event-id dedupe is not enough; view-credit and watch-segment dedupe are required.
- Component 3 uses idempotent/outbox publishing so dedupe cannot swallow events after a failed publish.
- Fraud and policy corrections are first-class streams and can emit negative ranking deltas.
- Hot-key salting must be combined by `videoId` before final top-k ranking.
- Component 4 topics and retention are explicit.

Initial implementation direction:

- Java 21
- Maven
- JUnit 5
- Production interfaces with local/in-memory adapters first
- Sliding windows using event-time buckets
- Metrics: `valid_views` and `watch_time_ms`
- Scopes: `global` and `country`
