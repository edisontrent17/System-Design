# Repository Instructions

## Project

This repository is for a production-oriented Java implementation of a YouTube top-k ranking system.

The system should model YouTube as a first-party platform with access to validated watch events. The serving API must read materialized ranking snapshots rather than computing rankings synchronously from raw events.

## Engineering Principles

- Discuss architecture and tradeoffs before adding implementation code.
- Prefer simple, explicit Java over clever abstractions.
- Keep production boundaries clear: ingestion, validation, aggregation, ranking, storage, and serving should be separate concepts.
- Do not claim internet-scale behavior from an in-memory prototype. Label prototype components clearly.
- Use deterministic tests for ranking behavior, windowing behavior, deduplication, and tie-breaking.
- Keep APIs versioned from the start.

## Initial Technology Direction

- Language: Java
- Build tool: Maven unless we choose otherwise before implementation.
- Test framework: JUnit 5.
- Initial prototype may use in-memory components behind interfaces.
- Production target should allow swapping in Kafka/Pulsar, Flink/Beam/Kafka Streams, Redis, and a durable snapshot store.

## Design Constraints

- Top-k requests should resolve a registered ranking definition and read a precomputed snapshot.
- Ranking snapshots must include freshness metadata such as `asOf`, `watermark`, and `rankQuality`.
- Watch events must be validated before they affect rankings.
- Ranking formulas must be versioned.
- Late events and correction snapshots must be considered in the design.
- Abuse, bot filtering, and duplicate event handling are part of the core system, not optional add-ons.

## Collaboration

- Before implementation, agree on the first production slice.
- Keep commits scoped and intentional.
- When changing behavior, add or update tests in the same change.
