# kafka

Notes and projects to learn how to utilize Apache Kafka

## Big Data End-to-End project Notes

Reference architecture:

gathering data from several platforms: twitter, MySQL, OpenWeather, anotehr web application etc.

These are data sources with open APIs.

These sources can push data into the Kafka messaging system. Kafka uses topics to organize data into buckets. And consumers can grab the data using Spark or other processes.

Spark allows for data processing at a large-scale.

You can also use Kafka with Zookeeper to handle the message traffic. This is becoming outdated though.

Once a message is consumed through Spark, you can write the data in an HDFS format. Then further querying can be done using Hive or Impala.

Once you have the queries setup, the data can be gathered using Presto (to get a low latency response). Then you can use Oozie or something similar to schedule the job (like Airflow).

## Apache Kafka and Python

As a developer, you'll either work on a brand new application, or be supporting a legacy application. No matter if something is new or old, it won't work in isolation. There are parts of the application talking to other applications.

Apache Kafka makes event-based communication between applications easy, fast, and reliable. Before kafka, events would be stored in databases and then passed into the application frontend. The application would be reading or writing events to the database and the application. This builds in a delay we can't accept in modern programming.

These event driven applications often work together as well.

Think about an event as a phone notification, and there are some important information that needs to be received immediately, not in 6 hours. For example, when you create an order on Uber Eats, the information is immediately conveyed to the restaurant staff, then once completed, the information gets passed to the driver. These have to happen in near real-time, because the information is most useful the quicker its delivered.

Kafka is the tool to make this communication reliable, efficient and swift.

### Kafka Log

Kafka stores data in a log called a **topic**. Every new record is written at the end of the log, and it is immutable, every event is stored, and any updates are stored as separate events.

These topics can be associated with one another. And these topics are implemented in a distributed system.

They store the topics across brokers multiple times. Kafka has a replication factor parameters that allow you to store multiple copies of the same topic on different nodes of the distributed system, called brokers. This way, if any one node goes down, the data isn't lost.

### Kafka Events

An Event is composed by a key and a value. These could be anything, but both things are needed for an event.

You can the key and value as extended JSON objects too.

You can make these small or large, and you can make the message as human readable, but this can make the message heavy on the brokers.

You can reduce the size of a message on the sending end, and then format it on the receiving end as well.

### Writing to Kafka

An application that writes to kafka is called a producer. These can write to any Kafka topic, including a python producer. The producer needs:

- the Hostname of the broker
- the port Kafka is listening on
- Authentication credentials
- Encoding the data

### Reading from Kafka

An application that reads from kafka is called a consumer.

A consumer reads event number zero then goes through each record in the topic, and then communicates back to Kafka that the a record was read.

So if a node goes down, the Kafka manager will know where the last node stopped, and can start the new node at the stopping point.

A consumer needs:

- the hostname and port
- Authentication
- Decoding
- Topic Name

### Demo

You can install Kafka in the cloud of your choice, or use a managed service like AWS or Aiven.

The instructor built Jupyter Notebooks using the Python package `kafka-python`.

#### Demo Notes

If you connect to a producer and don't go back in history, you will only get the records from the time you connected, not the previous records. You have to set up your consumer to read the older records if that's the behavior you want.

A consumer will hang once you start the function to check the producer for records. its listening, so other code won't run until the consumer has some records.

Also, you may not want to store data in Kafka forever. You can use topic retention polices to set up a time period that Kafka stores the records in a topic. You can keep the data until a topic reaches a certain time period, or a certain amount of data (10 GB for instance).

Kafka will allow you to host a lot of events, the events are stored in a topic, the topic is stored across brokers. This means you have to either define the amount of events based on the size of the disk of the broker, or you have to set the size of the disk of the broker by the size of events to store in the topic.

To have a huge topic, you'll have to have a huge disk size. But Kafka has topic partitions to avoid running into this size limit issue. This allows you to define specific events in partitions of the topic, and distribute these partitions on several brokers. Brokers store partitions, not the full topic. So the trade off is actually between the partitions and the disk size of the broker.

This lets you know that if you have smaller disk sizes, you will need more partitions, and also, with the replication factor policy, you won't lose any data.

### Selecting a Partition

The records with the same key end up in the same partition. You can also remove the key value, and when that happens, the records are distributed in a round-robin fashion.

When reading from a partition, you could read records in a different ordering than they were written. Ordering is only guaranteed in the partition, not through the full topic.

So partitions must be carefully chosen to not care about this global ordering.

With more partitions, you'll have more independent threads that can write in parallel, so that you can have many producers adding records, and multiple consumers reading. You may not want to read the same record twice, so Kafka assigns to each consumer an subset of the partitions of a topic, that are not overlapping.

#### Partitions Demo notes

Kafka has an admin client that you can use to make the partitions.

Showed how the key for the record sent by a producer does determine the partition the record is stored in, which changes which consumer reads the data.

### History of Events in Kafka

Kafka stores the events, and a consumer can read in the full list of events in a topic in Kafka. Because one consumer may want to read in events one by one, but for others it may be useful to read in the whole topic.

You can define consumer groups to segregate these two use cases of the data stored in Kafka. You can define these consumers as two different applications.

### Kafka Integration with Connect

If you're introducing kafka into your current systems, it can be useful to use Kafka connect to integrate Kafka with any source or target available.

Kafka can ingest data from databases and store them into topics. Kafka connect also supports sending topics to other consumers like into databases, or services.

You can also use Kafka to monitor existing applications. You can leave the applications in there current state but introduce some change detection scripts in Kafka connect to have the regular application support event driven processes.
