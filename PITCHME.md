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

# Structure

__4 Major Parts__

- Broke (Pipe)
- Producer (Out)
- Consumer (In)
- Message

+++

## Broker

- nodes of the cluster
- Kafkas backbone
- Data handling
- depends on Apache Zookeeper
- run a minimum of 3 in production

+++

## Producer

- produces a Message and sends it to the Broker
- can be a connection to a database
- SDKs for most programming languages

+++

## Consumer

- receives a Message from a Broker
- has no knowledge of the Producer
- can be scaled horizontally

+++

## Message

- is a key value pair
- the value contains the payload
- the key is used for routing

---

# Data Organization

- Data belong to Topics
- Topis are stored on Partitions
- Topics are replicated over Broker
- Partition Leader organize the data of a Topic

---

# END