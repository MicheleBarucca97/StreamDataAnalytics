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

![My Alt Text](/path/to/my/pic.jpg "My Optional Title Text")
