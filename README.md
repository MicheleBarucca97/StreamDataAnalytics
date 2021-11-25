# StreamDataAnalytics
Introduction
============
Project focused on the parsing of data coming from wikipedia using Kafka, in particular ksqldb

What is Kafka?
------------
The unit of data within Kafka is called **message**. A message is simply an array of bytes, which can also contain a bit of metadata called key, which is also a byte array. Depending on the use case, keys may be used or not, they provide a way to populate topics’ partitions in a more controlled manner. A message can also be referred to as a key-value pair; you can think of them as records in a traditional SQL database.

In most cases, messages need to have some structure that can easily be interpreted from other systems (schema). The most popular formats being used are JSON, XML and Avro.
Messages are categorized into different **topics**, in order to separate them based on some attribute. Topics can also be divided into partitions, which provides extra scalability and performance, as they can be hosted into different servers. You can think of topics as an append-only log, which can only be read from beginning to end. In the SQL world, topics would be our tables.

There are two types of clients; **publishers** and **consumers**. As their names imply, **publishers** send messages to topics and **consumers** read them.
A Kafka node is called a **broker**. Brokers are responsible for acquiring messages from producers, storing them in disk and responding to requests of consumers. Many brokers form a cluster. Partitions can only be owned by one broker into a cluster called the leader.

![My Alt Text](https://github.com/MicheleBarucca97/StreamDataAnalytics/blob/main/Images/kafka_architecture.jpg "Kafka Architecture")

Some of the key features that make Apache Kafka a great product:

+ Multiple producers can publish messages at the same time to the same topics.
+ Multiple consumers can read data independently from others or in a group of consumers sharing a stream and ensuring that each message will be read only once across a group.
+ Retention, data published to cluster can persist in disk according to given rules.
+ Scalability, Kafka is designed to be fully scalable as it is a distributed system that runs on multiple clusters of brokers across different geographical regions, supporting multiple publishers and consumers.
+ Performance, on top of the features mentioned above, Kafka is extremely fast even with a heavy load of data, providing sub-second latency from publishing a message until it is available for consuming.

Case study 
=========
The use case is a Kafka event streaming application for real-time edits from real Wikipedia pages. Wikimedia Foundation has IRC channels that publish edits happening to real wiki pages in real time. To bring the edits from Wikipedia to the Kafka cluster I decide to use Python, in which there are multiple libraries available for usage, in this spceific case I refere to the library **ksql-python**: a python wrapper for the KSQL REST API. Easily interact with the KSQL REST API using this library. The general set-up to use suche library is the following:

```python
from ksql import KSQLAPI
client = KSQLAPI('http://ksql-server:8088')
```

To have more information about the following library I redirect you at [the following link](https://libraries.io/pypi/ksql).
This demo uses ksqlDB and a Kafka Streams application for data processing. This is a Docker environment and has all services running on one host. Also notice that the operative system in which the application was built is Ubuntu 20.04. 

How to run the demo  
=========
The first step to run the demo is to enable some permission, since the default permissions on ```/var/run/docker.sock``` is generally owned by user root and group docker, with mode 0660 (read/write permissions for owner and group, no permissions for others). So in order to use docker, first of all you have to run a small bash script tath you can find in the repos under the name ```script```.

The docker-compose file
---------
You have to set up the Kafka environment, this is easely done by running the docker-compose file with the flag ```-d```, in this way it runs in background and you can continue to use the same terminal window, the complete command is:   ```docker-compose up -d```.

You can define a ksqlDB application by creating a stack of containers. A stack is a group of containers that run interrelated services.The minimal ksqlDB stack has containers for Apache Kafka, ZooKeeper ([What is Zookeeper](https://zookeeper.apache.org/)) and ksqlDB Server. More sophisticated ksqlDB stacks can have Schema Registry, Connect, and other third-party services, like Elasticsearch.
Stacks that have Schema Registry can use Avro- and Protobuf-encoded events in ksqlDB applications. Without Schema Registry, your ksqlDB applications can use only JSON or delimited formats (which is our case).

The usual configuration is the following:

```docker-compose
---
version: '2'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:6.2.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-kafka:6.2.0
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

  ksqldb-server:
    image: confluentinc/ksqldb-server:0.21.0
    hostname: ksqldb-server
    container_name: ksqldb-server
    depends_on:
      - broker
    ports:
      - "8088:8088"
    environment:
      KSQL_LISTENERS: http://0.0.0.0:8088
      KSQL_BOOTSTRAP_SERVERS: broker:9092
      KSQL_KSQL_LOGGING_PROCESSING_STREAM_AUTO_CREATE: "true"
      KSQL_KSQL_LOGGING_PROCESSING_TOPIC_AUTO_CREATE: "true"

  ksqldb-cli:
    image: confluentinc/ksqldb-cli:0.21.0
    container_name: ksqldb-cli
    depends_on:
      - broker
      - ksqldb-server
    entrypoint: /bin/sh
    tty: true
```

To see that everything went smoothly you can type on the terminal ```docker ps -a``` and see that all the images are running.

Start the ksqlDB CLI
---------
When all of the services in the stack are Up, run the following command to start the ksqlDB CLI and connect to a ksqlDB Server.
Run the following command to start the ksqlDB CLI in the running ksqldb-cli container:

```docker exec ksqldb-cli ksql http://primary-ksqldb-server:8088```

In the Git-repos you can find a bash script containing this command, so in the terminal you can simply run ```./launch-ksql```:

```
                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =        The Database purpose-built       =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2020 Confluent Inc.

CLI v0.22.0, Server v0.22.0 located at http://primary-ksql-server:8088

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>
```

With the ksqlDB CLI running, you can issue SQL statements and queries on the ksql> command line.
Before we see how to send data to ksqlDB, we need to execute some commands ksqlDB CLI. The first command we have to run is:
```SET 'auto.offset.reset' = 'earliest';```
This command allows to return the queries always from the beginning of the data streams, this is used for debugging purposes.

Next, we have to create the stream in which we will insert the data
```
CREATE STREAM Wikipedia_STREAM (domain VARCHAR,
  namespaceType VARCHAR,
  title VARCHAR,
  timestamp VARCHAR,
  userName VARCHAR,
  userType VARCHAR,
  oldLength INTEGER,
  newLength INTEGER)
  WITH (
  kafka_topic='Wikipedia_topic',
  value_format='json',
  partitions=1);
```
To see if everything worked well, we can do two things:
1. ```show topics;``` -> to see if the topic has been created, in our case it is 'Wikipedia_topic'
2. ```INSERT INTO Wikipedia_STREAM (domain, namespaceType, title, timestamp, userName, userType, oldLength, newLength) VALUES ('www.wikidata.org', 'main namespace', 'Q109715322', '2021-11-24T16:59:10Z', 'SuccuBot', 'bot', 1486, 1850);``` -> in this way we see that data are inserted properly

Python application
---------
