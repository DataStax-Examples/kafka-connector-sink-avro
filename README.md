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

Start the DataStax Kafka Connector using the Kafka Connect REST API
```
curl -X POST -H "Content-Type: application/json" -d @connector-config.json "http://localhost:8083/connectors"
```

Confirm that the rows were written in Cassandra
```
docker exec -it cassandra cqlsh
```
```
select * from kafka_examples.avro_udt_table limit 10;
```
