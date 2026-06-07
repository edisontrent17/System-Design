# YouTube Sliding Top-K Rankings

Production-oriented design notes for a YouTube-style top-k ranking system over sliding windows.

## GitHub Pages

Architecture page:

https://edisontrent17.github.io/System-Design/

## Core Idea

The serving API should not compute top-k rankings from raw events. It should read materialized ranking snapshots produced by a streaming pipeline.

```text
Playback Events
  -> Validation / Deduplication / Fraud Filtering
  -> Kafka or PubSub
  -> Sliding Window Aggregation
  -> Shard-Local Top-K
  -> Global Top-K Merge
  -> Ranking Snapshot Store
  -> Redis or Edge Cache
  -> Top-K API
```

## Ranking Contract

Counts are exact for processed, validated events up to the published watermark. Raw playback starts, valid views, and public counters are intentionally different concepts.

Initial implementation direction:

- Java 21
- Maven
- JUnit 5
- Production interfaces with local/in-memory adapters first
- Sliding windows using event-time buckets
- Metrics: `valid_views` and `watch_time_ms`
- Scopes: `global` and `country`
