# Kafka

## What is Apache Kafka

> Apache Kafka is an open-source __stream-processing__ software platform developed by the Apache Software Foundation, written in Scala and Java. The project aims to provide a unified, high-throughput, low-latency platform for handling __real-time__ data feeds. Its storage layer is essentially a "massively scalable pub/sub message queue designed as a distributed transaction log," making it highly valuable for enterprise infrastructures to process streaming data.

_Wikipedia [Apache Kafka](https://en.wikipedia.org/wiki/Apache_Kafka)_

Apache Kafka is an ESB like system that provides near realtime message handling of giant amount of messages. There some limitations that need to be kept in mind when implementing a flow with Apache Kafka.

- messages need to be small 10 kB max (best practice)
- plan for resilient
- plan for how data is distributed

## Topics

Kafka uses topics as an address for a message. All messages are stored for a configured time (standard is 1 week).

## Partitions

Partitions are used to achieve horizontal scaling by spreading a topic over multiple partitions. On a partition the order of the messages is guaranteed.

- based on modulo hash value of the key and the amount of partitions
- when the key changes or the amount of partitions changes Kafka needs to make repartitioning that will block the complete process

## Offset

The offset describes the position of a message for a topic on a specific partition.

## Message

A message that is published to Kafka is immutable. It can't be changed later.

## Broker

The brokers are the heart of Kafka. For production and production near systems a minimum of 3 brokers is recommended. To access the cluster only one needs to be reached (bootstrap broker). The reached broker will provide the client with all information of the cluster. Each broker is identified by an id. It contains some of the topics partitions.

## Replication Factor

The factor describes how much redundant copies are spread over the brokers.

## Partition Leader

Only one leader exists for a partition. ISR (In-Sync-Replica) is used to copy all messages from the leader to the follower. Only the leader will consume and publish messages.

## Producer

Producers send data as a Key-Value-Pair to the broker cluster. The producer has three options on how to acknowledge of data writes:

1. at-most-once: Standard configuration, 0 to multiple delivery
	2. fast, no garenty
2. at-least-once: 1 or more deliveries
	3. slower, handling for multiple deliveries
3. exactly-once: only 1 delivery
	4. very slow

## Consumer

A consumer connects to a broker and subscribes for a specific topic. All message handling is done by Kafka.

## Consumer Groups

Consumers in a Consumer Group reads data from an exclusive partition.  This might be one or more partitions, but two consumers can't read from the same partition when they are in the same Consumer Group. When there are more consumers then partitions, these partition will idle.

## Consumer Offset

Kafka stores the offset of each Consumer Group. The offset describes the point to which messages have been processed. This mechanism creates resilience.

## Key

The key of a message is used to decide to which partition a message will go. To have all messages go to the same partition the same key must be used. It is important to know that based on the keys hash value, a modulo calculation with the amount of partitions for that topic is done. This should result in an almost balanced distribution.

To use a round robin mode, a null value for the key can be passed.

[Delivery modes for Kafka - Java examples](https://dzone.com/articles/kafka-clients-at-most-once-at-least-once-exactly-o)

## Value

The value is the payload that will be the content of the message. Here serializable formats like Json or Avro are used.

## Zookeeper

Zookeeper sits between the broker and the harddrive, manages all data handling and helps keeping the cluster up and running. Kafka depends on Zookeeper. There are possibilities to [run Kafka on k8s without Zookeeper](https://banzaicloud.com/blog/kafka-on-etcd/).

Zookeeper follows the leader follower system, so only one node will be the leader and all the others are follower.

# Running

## Local development

With Docker Kafka can be executed locally.

```
$> docker run -it --rm \
				-p 2181:2181 \
				-p 3030:3030 \
				-p 8081-8083:8081-8083 \
				-p 9581-9585:9581-9585 \
				-p 9092:9092 \
				-e ADV_HOST=127.0.0.1 \
				landoop/fast-data-dev
```

After downloading and starting the container the overview dashboard from landoop can be fund under `localhost:3030`.

To interact with the Kafka cluster, the image from landoop comes with the complete tool suite.

```
$> docker run -it --rm --net=host landoop/fast-data-dev bash
```

This command will bring you to the console of the container with all tools defined in the global namespace.

## Creating a new Topic

Use the jumpbox to run a command.

```
$> kafka-topics \
	--zookeeper 127.0.0.1:2181 \
	--create \
	--topic name_of_the_topic \
	--partitions 3 \ 
	--replication-factor 1
	
$> kafka-topics --zookeeper 127.0.0.1:2181 --list

$> kafka-topics --zookeeper 127.0.0.1 \
	--describe \
	--topic name_of_the_topic
```

This creates a new topic on the Kafka cluster.

Best practice is to create a new topic before sending a new message to that topic.

## Send a Message

To send a message use the cli tools.

```
$> kafka-console-producer \
	--broker-lis 127.0.0.1:9092 \
	--topic name_of_the_topic
```

After firing that command, message can be send directly from the console.

## Consume a Message

To consume a message from the cli the cli tools must be attached to a broker.

```
$> kafka-console-consumer \
	--bootstrap-server 127.0.0.1:9092 \
	--topic name_of_the_topic
```

## Guidlines

2,000 to 4,000 partitions per broker. All in all the overall number of partitions should be lower than 20,000.

To calculate the amount of partitions: 1 or 2 x number of broker, max 10 partitions.

Replications should be 2 or max 3. If there are more replicas around, the amount of disk space increases by times of this amount and longer replications. Best practice is 3.

## Segments

Partitions contains of segments. These segments have an offset and a range and an id. The last segment has no upper bound, because it is active and still increasing.

Segments are created based on time and size. Both values are configurable. Depending on which is hit at first, the segment is created. Segments have two indexes. An offset to a position and a timestamp to the offset index.

By default the file size of a segment is 1GB. Smaller size would lead to more often log compaction and Kafka needs to have more files open. This might lead to an error. The default value for the timestamp is 1 week.

## Clean Up

A cleanup happens for segments. If segments are smaller a cleanup happens more often, but will consume a resources (Ram, CPU).

```
log.cleaner.backoff.ms=15000
```

### Policy 1

Kafaka deletes data based on age, default is 1 week for all user topics. It is possible to configure a second parameter based on file size. Default here is -1 == infinite and per partition.

```
log.retention.hours: 168 // one week default
log.retention.byte: -1 // infinite
log.cleanup.policy=delete
```

### Policy 2

Here deletes are based on the key. Kafka will keep no history data, when a new event comes in for an already existing key.

```
log.cleanup.policy=compact
```

## Log Compression

```
compression.type: producer // (gzip, snappy, lz4, uncompressed, producer)
```

Compress only text messages. Byte formats like Parquet, Protobuf, Avro what have any size reduction when compressed.

# Kafka Streams

Kafka Streams are constructed out of processors. The processors are chained together, starting with a producer, an origin and ends in a sink. All events that are part of theses topics will flow through it.

## KStream

The KStream is the pipe that translate one message to the next processing point.

- inserts
- equivalent to a log
- infinite
- unbound data stream

## KTable

- updates non null key
- deletes in null key
- lika a SQL table
- they are compacted topics

## When to one over the other

### KStream

- topic is not compacted
- partial updates
- keep the complete history

### KTabele

- topic is compacted / aggregated
- the results needs to be in aSQL like structure

## Stateful and Stateless

Processing transformations can be stateful or stateless, depending on the operation. When the calculation does not need to know anything about what had happen before, it is stateless. But when the operation is depending on a result from before it becomes stateful.

## Processing Functions for KStreams

### map

- affects both keys and values
- triggers a repartition
- KStream only
- one to one record

### mapValues

- is only affecting values
- does not change key
- does not trigger a repartition
- for KStream and KTables
- one to one record

### filter filterNot

- one to zero or many records
- does not change key / value
- does not trigger a repartition

### flatMapValues

- one to zero or many records
- does not change keys
- does not trigger a repartition
- for KStreams only

### flatMap

- changes keys
- triggers a repartition
- for KStreams only

### branch

- split a KStream based on predicates
- predicates are evaluated in order, when it doesn't match the record is dropped
- the results are multiple KStreams

### selectKey

- assigns a new key to the record
- marks data for repartitioning
- best practice is to isolate the transformation to know exactly where the partitioning happens

### peek

- read a key value pair
- run any action with these data
- returns the same event stream
- events are unchanged
- test

## Terminal Functions for KStreams and KTables

### to

- writes the result to a topic
- for KStream and KTable

### through

- writes to a topic and receives the result back
- for KStream and KTable

## Log Compaction

- only the last state is kept on the stream
- null key values removes a key-value pair from the stream
- improves space a lot
- log compaction needs to be done explicit

## Development

During development it is good to disable Kafkas cache options by setting the buffer to zero.

```
StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG = "0"
```

# Glossary

Broker
Consumer
Producer
Zookeeper
Kafka Connect Source or Sink
Partition
Topic
Kafka Streams
Confluent
Avro aka Schema Registry
Offset
Replication Factor

# Tooling

- Topics UI (Landoop): view the topics content
- Schema UI (Landoop): explore the schema registry
- Connect UI (Landoop): create and monitor Connect tasks
- Kafka Manager (Yahoo): Overall Kafka Cluster Management
- Burrow (LinkedIn): Kafka Consumer Lag Checking
- Exhibitor (Netflix): Zookeeper Configuration, Monitoring, Backup
- Kafka Monitor (LinkedIn): Cluster health monitoring
- Kafka Tools (LinkedIn): Broker and topics administration tasks
- Kafkat (AirBnB): Broker and topics administration tasks
- JMX Dump: Dump JMX metrics from Broker

