# Kafka Connect sink connector for JDBC

Thhis is a customised version of the original IBM code. Changes are:

1) Only works for PostgreSQL
2) Assumes the Kafka record value is a JSON object
3) If table doesn't exist, creates it with 2 columns, id (integer) and json_object (jsonb)
4) Inserts the JSON object into the json_object columns for every record.

Possible improvements: Could allow custom column names.

Notes: 

1 Seems to well reliably, if the Kafka record isn't valid JSON then PostgreSQL throws an exception, the record is ignored, and the connector keeps on going with the next record.

2 table.name.format in the config file MUST have have a value in the format "schemaName.tableName" - i.e. a schema is mandatory

3 No other changes to the configuration are required.

From here, the original documentation (with testing removed as it no longer accepts schema and payload format records).

kafka-connect-jdbc-sink is a [Kafka Connect](http://kafka.apache.org/documentation.html#connect) sink connector for copying data from Apache Kafka into a JDBC database.

The connector is supplied as source code which you can easily build into a JAR file.

## Installation

1. Clone the repository with the following command:

```bash
git@github.com:ibm-messaging/kafka-connect-jdbc-sink.git
```

2. Change directory to the `kafka-connect-mq-sink` directory:

```shell
cd kafka-connect-mq-sink
```

3. Build the connector using Maven:

```bash
mvn clean package
```


4. Setup a local zookeeper service running on port 2181 (default) 

5. Setup a local kafka service running on port 9092 (default)

6. Setup a local rabbitmq service running on port 15672 (default)

7. Copy the compiled jar file into the `/usr/local/share/java/` directory:

```bash
cp target/kafka-connect-jdbc-sink-0.0.1-SNAPSHOT-jar-with-dependencies.jar /usr/local/share/java/
```

8. Copy the `connect-standalone.properties` and `jdbc-sink.properties` files into the `/usr/local/etc/kafka/` directory.

```bash
cp config/* /usr/local/etc/kafka/
```

9. Go to the kafka installation directory `/usr/local/etc/kafka/`:

```bash
cd /usr/local/etc/kafka/
```

10. Set the CLASSPATH value to `/usr/local/share/java/` as follows:

```bash
export CLASSPATH=/usr/local/share/java/
```

## Configuration

1. Create a target kafka topic named `kafka_test`:

```shell
kafka-topics --create --topic kafka_test --partitions 3 --replication-factor 1 --zookeeper 127.0.0.1:2181
```

2. Set up a JDBC database with an accessible URL and Port Number as well as a user with read/write privileges.

Setting up this database involves creating the database, creating user with password and proper access privileges.

Below are some of the commands involved in setting up databases using postgresql:

```bash
create user {username};
create database {dbname};
grant all privileges on database {dbname} to {username};

\l - list databse
\du is to verify user roles
\c to select database
```

Below are some of the commands involved in setting up databases using db2 using a docker image:

```bash
1. docker network create DB2net
2. mkdir db2
3. cd db2
4. docker run -it -d --name mydb2 --privileged=true -p 50000:50000 -e LICENSE=accept -e DB2INST1_PASSWORD=<Access Password> -e DBNAME=db2 -v "$PWD":/database --network DB2net ibmcom/db2
5. docker logs -f mydb2
	# make sure all 4 tasks are completed and
	# (*) All databases are now active
6. docker exec -it mydb2 bash -c "su - db2inst1"
7. db2
8. create db kafka_test
9. connect to kafka_test
10. list tables
11. select * from company
```

Download the dc2jcc.jar file from the following url: https://www.ibm.com/support/pages/db2-jdbc-driver-versions-and-downloads and place it into the jar classpath `/usr/local/share/java/`.


3. Open up the `config\jdbc-connector.json` file using the command below:

```bash
vi config\jdbc-connector.json
```

4. Set the following values in the `config\jdbc-connector.json` file:

```bash
    "connection.url": <CONNECTION_URL_OF_YOUR_DATABASE>,   (ie: "jdbc:postgresql://127.0.0.1:5432/postgres")
    "connection.user": <CONNECTION_USER>,                  (ie: "newuser")
    "connection.password": <CONNECTION_PASSWORD>,          (ie: "test")
    "table.name.format": <DATABASE_TABLE_NAME>             (ie: "company")
```

## Running in Standalone Mode

Run the following command to start the sink connector service in standalone mode:

```bash
connect-standalone connect-standalone.properties jdbc-sink.properties
```

## Running in Distributed Mode

1. In order to run the connector in distributed mode you must first register the connector with
Kafka Connect service by creating a JSON file in the format below:

```json
{
  "name": "jdbc-sink-connector",
  "config": {
    "connector.class": "com.ibm.eventstreams.connect.jdbcsink.JDBCSinkConnector",
    "tasks.max": "1",
    "topics": "kafka_test",
    "connection.url": "jdbc:postgresql://127.0.0.1:5432/postgres",
    "connection.user": "newuser",
    "connection.password": "test",
    "connection.ds.pool.size": 5,
    "insert.mode.databaselevel": true,
    "table.name.format": "company"
  }
}
```

A version of this file, `config/jdbc-connector.json`, is located in the `config` directory.  To register
the connector do the following:

1. Run the following command to the start the source connector service in distributed mode:

```bash
connect-distributed connect-distributed.properties
```

2. Run the following command to register the connector with the Kafka Connect service:

```bash
curl -s -X POST -H 'Content-Type: application/json' --data @config/jdbc-connector.json http://localhost:8083/connectors
```

You can verify that your connector was properly registered by going to `http://localhost:8083/connectors` which 
should return a full list of available connectors.  This JSON connector profile will be propegated to all workers
across the distributed system.  After following these steps your connector will now run in distributed mode.

## Issues and contributions

For issues relating specifically to this connector, please use the [GitHub issue tracker](https://github.com/ibm-messaging/kafka-connect-jdbc-sink/issues). If you do want to submit a Pull Request related to this connector, please read the [contributing guide](CONTRIBUTING.md) first to understand how to sign your commits.


## License

Copyright 2020 IBM Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    (http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.The project is licensed under the Apache 2 license.
