# Apache Kafka

+++

## Kafka Basics

Know the basics before running a geo redundant cluster live.

+++

## What is Apache Kafka?

> Apache Kafka is an open-source __stream-processing__ software platform developed by the Apache Software Foundation, written in Scala and Java. The project aims to provide a unified, high-throughput, low-latency platform for handling __real-time__ data feeds. Its storage layer is essentially a "massively scalable pub/sub message queue designed as a distributed transaction log," making it highly valuable for enterprise infrastructures to process streaming data.

_Wikipedia [Apache Kafka](https://en.wikipedia.org/wiki/Apache_Kafka)_

+++

## Use cases

- Streaming data application
- Microservice Architecture
- connecting data processing stacks (Flink, Spark, Storm)
- using different data storage system (graph, SQL, NoSQL)

---

## Structure

__4 Major Parts__

- Broke (Pipe)
- Producer (Out)
- Consumer (In)
- Message

+++

### Broker

- nodes of the cluster
- Kafkas backbone
- Data handling
- depends on Apache Zookeeper
- run a minimum of 3 in production

+++

### Producer

- produces a Message and sends it to the Broker
- can be a connection to a database
- SDKs for most programming languages

+++

### Consumer

- receives a Message from a Broker
- has no knowledge of the Producer
- can be scaled horizontally

+++

### Message

- is a key value pair
- the value contains the payload
- the key is used for routing

---

## The Kafka Story

- a _Producer_ sends a _Message_
- a _Message_ is sent to a _Topics_
- _Messages_ are stored in _Segments_ on a _Topic_
- _Topics_ are stored on _Partitions_
- _Topics_ are _replicated_ over _Broker_
- _Partition Leader_ organize the data of a _Topic_
- the _Message_ is sent to a _Consumer_
- the _Consumer_ _commits_ the _Offset_ to the _Broker_

---

### the flow of sending a Message

Image

// Show a message that moves through the system (simplified)

---

### The Architecture of Kafka

Image

// Show the boxed design of Kafka

+++

## Demo

---

### What you will see

- running a broker (single mode)
- creating a new topic
- send a message
- consume a message

// landoop session

+++

## Guidelines

- use max. 4,000 per broker
- use not more than 20,000 partitions
- use max 10 partitions per topic
- use 3 replicas (avoid split brains)

+++

## End of Life

- cleanups happen per Segment
- more often cleanups will increase Ram and CPU
- Kafka uses Cleanup Policies

```
log.cleaner.backoff.ms=15000
```

---

### Cleanup Policy 1

- deletion based on time (default 1 week)
- additional configurable file size (default -1 // infinite)

```
log.retention.hours: 168 // one week default
log.retention.byte: -1 // infinite
log.cleanup.policy=delete
```

---

### Cleanup Policy 2

- deletion for updated keys

```
log.cleanup.policy=compact
```

### Compression

- logs can be compressed
- only for text messages

```
compression.type: producer // (gzip, snappy, lz4, uncompressed, producer)
```

## END