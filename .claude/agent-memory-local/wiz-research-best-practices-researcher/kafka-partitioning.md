# Kafka Partitioning Mechanics (Researched 2026-03-04)

## Authoritative Sources Used
- Apache Kafka Protocol docs: https://kafka.apache.org/41/design/protocol/
- Confluent Producer docs: https://docs.confluent.io/platform/current/clients/producer.html
- Producer Config reference: https://kafka.apache.org/42/configuration/producer-configs/
- KIP-480 (Sticky Partitioner): https://cwiki.apache.org/confluence/display/KAFKA/KIP-480:+Sticky+Partitioner
- KIP-794 (Strictly Uniform Sticky): https://cwiki.apache.org/confluence/display/KAFKA/KIP-794:+Strictly+Uniform+Sticky+Partitioner

## Critical Finding: Client Decides, Not Broker
Official Kafka protocol docs state verbatim: "Kafka clients directly control this assignment, the brokers themselves enforce no particular semantics of which messages should be published to a particular partition."

## Metadata Flow
1. Producer connects to bootstrap broker(s)
2. Sends MetadataRequest → receives full cluster topology (brokers, topics, partition count, leaders)
3. Producer caches this metadata (refreshed by metadata.max.age.ms, default 5 min)
4. Partitioner uses partition count from metadata for key hashing (murmur2 % num_partitions)

## Default Partitioner Priority
1. Explicit partition specified → use it
2. Key present → murmur2(key) % num_partitions
3. No key → Sticky (Kafka 2.4+) or Round-Robin (< 2.4)

## KIP-480 (Kafka 2.4): Sticky Partitioner
- Sticks to one partition until batch is full or linger.ms expires, then switches randomly
- ~50% latency reduction vs round-robin

## KIP-794: Strictly Uniform Sticky (current default)
- Fixes KIP-480's flaw: slower brokers were getting more messages
- Weighted toward faster brokers (partitioner.adaptive.partitioning.enable)

## Two Producers, Different Strategies
- Broker is completely agnostic — accepts any valid partition number
- No enforcement at broker level
- Risk: same key can land in different partitions, breaking per-key ordering guarantees
- Best practice: standardize partitioning strategy across all producers writing to a topic
