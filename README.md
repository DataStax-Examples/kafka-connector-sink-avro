# Ingest Avro from Kafka to Cassandra
This example shows how to ingest Avro records from [Apache Kafka](https://kafka.apache.org/) to a table in the [Apache Cassandra](https://cassandra.apache.org/) database using the [DataStax Apache Kafka Connector](https://docs.datastax.com/en/kafka/doc/index.html).

Contributor(s): [Chris Splinter](https://github.com/csplinter), [Tomasz Lelek](https://github.com/tomekl007)

Have Questions? We're here to help: https://community.datastax.com/

Want to learn more about the DataStax Kafka Connector? Take a free, [short course on DataStax Academy](https://academy.datastax.com/resources/getting-started-datastax-apache-kafka%E2%84%A2-connector)

Looking for a fully-managed service built on Apache Cassandra? Try DataStax Astra for free: https://astra.datastax.com/

## Objectives
- How to ingest Avro records from Kafka to Cassandra
- How to use docker and docker-compose to quickly set up an environment with Zookeeper, Kafka Brokers, Kafka Connect, Confluent Schema Registry and Cassandra

## Project Layout
- [Dockerfile-connector](Dockerfile-connector): Dockerfile to build an image of Kafka Connect with the DataStax Kafka Connector installed.
- [Dockerfile-producer](Dockerfile-producer): Dockerfile to build an image for the producer contained in this repository.
- [docker-compose.yml](docker-compose.yml): Uses [Confluent](https://www.confluent.io/) and DataStax docker images to set up Zookeeper, Kafka Brokers, Kafka Connect, Confluent Schema Registry, Cassandra, and the producer container.
- [connector-config.json](connector-config.json): Configuration file for the DataStax Kafka Connector to be used with the distributed Kafka Connect Worker.
- [producer](producer/): Contains the Kafka Avro Producer to write records to Kafka. Uses the AvroSerializer for the Kafka record key and record value.

## How this works
After running the docker and docker-compose commands, there will be 6 docker containers running, all using the same docker network.

After writing records to the Kafka Brokers, the Kafka Connector will be started which will start the stream of records from Kafka to Cassandra, writing the records to a table in the database.

## Setup & Running
### Prerequisites
- Docker: https://docs.docker.com/v17.09/engine/installation/
- Docker Compose: https://docs.docker.com/compose/install/

### Setup
Clone this repository
```
git clone https://github.com/DataStax-Examples/kafka-connector-sink-avro.git
```

Go to the directory
```
cd kafka-connector-sink-avro
```

Build the DataStax Kafka Connector image
```
docker build . -t datastax-connect -f Dockerfile-connector
```

Build the Avro Java Producer image
```
docker build . -t kafka-producer -f Dockerfile-producer
```

Start Zookeeper, Kafka Brokers, Kafka Connect, Confluent Schema Registry, Cassandra, and the producer containers
```
docker-compose up -d
```

### Running
Now that everything is up and running, it's time to set up the flow of data from Kafka to Cassandra.

Create the Kafka Topic named `avro-stream` that the connector will read from.
```
docker exec -it kafka-broker bash
```
```
kafka-topics --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 10 --topic avro-stream --config retention.ms=-1
```

Create the Cassandra table that the connector will write to. Start the cqlsh shell and then copy and paste the contents of [`schema.cql`](schema.cql)
```
docker exec -it cassandra cqlsh
```

Write 1000 records to Kafka using the Avro Java Producer
```
docker exec -it kafka-producer bash
```
```
mvn clean compile exec:java -Dexec.mainClass=avro.AvroProducer -Dexec.args="avro-stream 1000 broker:29092 http://schema-registry:8081"
```
There will be many lines of output in your console as Maven pulls down the dependencies. The following output means that it completed successfully
```
2020-08-04 13:59:25.990 [avro.AvroProducer.main()] INFO  - Completed loading 1000/1000 records to Kafka in 1 seconds
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 32.195 s
[INFO] Finished at: 2020-08-04T13:59:25+00:00
[INFO] Final Memory: 35M/248M
[INFO] ------------------------------------------------------------------------
```

Execute the following command from the machine where docker is running to start the connector using the Kafka Connect REST API
```
curl -X POST -H "Content-Type: application/json" -d @connector-config.json "http://localhost:8083/connectors"
```

Confirm that the rows were written in Cassandra
```
docker exec -it cassandra cqlsh
```
```
select * from kafka_examples.avro_udt_table limit 1;
```
You should see the following output:
```
cqlsh> select * from kafka_examples.avro_udt_table limit 1;

 id  | udt_col0                                                                                                                                                                   | udt_col1                                                                                                                                                                   | udt_col2                                                                                                                                                                   | udt_col3                                                                                                                                                                   | udt_col4                                                                                                                                                                   | udt_col5                                                                                                                                                                   | udt_col6                                                                                                                                                                   | udt_col7                                                                                                                                                                   | udt_col8                                                                                                                                                                   | udt_col9
-----+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 769 | {segment0_0: '0', segment0_1: '1', segment0_2: '2', segment0_3: '3', segment0_4: '4', segment0_5: '5', segment0_6: '6', segment0_7: '7', segment0_8: '8', segment0_9: '9'} | {segment1_0: '0', segment1_1: '1', segment1_2: '2', segment1_3: '3', segment1_4: '4', segment1_5: '5', segment1_6: '6', segment1_7: '7', segment1_8: '8', segment1_9: '9'} | {segment2_0: '0', segment2_1: '1', segment2_2: '2', segment2_3: '3', segment2_4: '4', segment2_5: '5', segment2_6: '6', segment2_7: '7', segment2_8: '8', segment2_9: '9'} | {segment3_0: '0', segment3_1: '1', segment3_2: '2', segment3_3: '3', segment3_4: '4', segment3_5: '5', segment3_6: '6', segment3_7: '7', segment3_8: '8', segment3_9: '9'} | {segment4_0: '0', segment4_1: '1', segment4_2: '2', segment4_3: '3', segment4_4: '4', segment4_5: '5', segment4_6: '6', segment4_7: '7', segment4_8: '8', segment4_9: '9'} | {segment5_0: '0', segment5_1: '1', segment5_2: '2', segment5_3: '3', segment5_4: '4', segment5_5: '5', segment5_6: '6', segment5_7: '7', segment5_8: '8', segment5_9: '9'} | {segment6_0: '0', segment6_1: '1', segment6_2: '2', segment6_3: '3', segment6_4: '4', segment6_5: '5', segment6_6: '6', segment6_7: '7', segment6_8: '8', segment6_9: '9'} | {segment7_0: '0', segment7_1: '1', segment7_2: '2', segment7_3: '3', segment7_4: '4', segment7_5: '5', segment7_6: '6', segment7_7: '7', segment7_8: '8', segment7_9: '9'} | {segment8_0: '0', segment8_1: '1', segment8_2: '2', segment8_3: '3', segment8_4: '4', segment8_5: '5', segment8_6: '6', segment8_7: '7', segment8_8: '8', segment8_9: '9'} | {segment9_0: '0', segment9_1: '1', segment9_2: '2', segment9_3: '3', segment9_4: '4', segment9_5: '5', segment9_6: '6', segment9_7: '7', segment9_8: '8', segment9_9: '9'}

(1 rows)
```
