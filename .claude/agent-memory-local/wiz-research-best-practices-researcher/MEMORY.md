# Research Agent Memory

## Project Context
- Project: fantastic_articles — article repository
- Stack: Go (backend), TypeScript (frontend/tooling)
- Working directory: /home/adi/proyects/fantastic_articles

## Kafka Research (2026-03-04)
Researched Kafka partitioning mechanics. Key findings preserved in kafka-partitioning.md.
Summary: Partition assignment is entirely client-side (producer decides, not broker). Producer fetches cluster metadata (MetadataRequest) from bootstrap brokers before sending. Official Kafka protocol docs state this explicitly.
