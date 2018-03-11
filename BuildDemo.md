We will use a distributed platform that turns our existing databases into event streams, so applications can see and respond immediately to each row-level change in the databases. Our platform (Debezium) is built on top of Apache Kafka and provides Kafka Connect compatible connectors that monitor specific database management systems. This records the history of data changes in Kafka logs, from where the application consumes them. This makes it possible for the application to easily consume all of the events correctly and completely. Even if the application stops (or crashes), upon restart it will start consuming the events where it left off so it misses nothing.

In Production: 
Production environments, require running multiple instances of each service to provide the performance, reliability, replication, and fault tolerance. 
This can be done with a platform like OpenShift and Kubernetes that manages multiple Docker containers running on multiple hosts and machines on dedicated hardware.

Caveat: You cannot remove a row that is referenced by a foreign key.

STEPS:

Start Zookeeper
$ docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:0.7
Zookeeper is ready and listening on port 2181

Start Kafka
$ docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:0.7
If we wanted to connect to Kafka from outside of a Docker container, then we’d want Kafka to advertise its address via the Docker host, which we could do by adding -e ADVERTISED_HOST_NAME= followed by the IP address or resolvable hostname of the Docker host, which on Linux or Docker on Mac this is the IP address of the host computer (not localhost).

Start a MySQL database
$ docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:0.7

Start a MySQL command line client
$ docker run -it --rm --name mysqlterm --link mysql --rm mysql:5.7 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

switch to the "inventory" database
mysql> use inventory;
mysql> show tables;

+---------------------+
| Tables_in_inventory |
+---------------------+
| customers           |
| orders              |
| products            |
| products_on_hand    |
+---------------------+
4 rows in set (0.00 sec)

mysql> SELECT * FROM customers;

Start Kafka Connect
$ docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:0.7

Using the Kafka Connect REST API
The Kafka Connect service exposes a RESTful API to manage the set of connectors, so let’s use that API using the curl command line tool. Because we mapped port 8083 in the connect container

Open a new terminal, and use it to check the status of the Kafka Connect service
$ curl -H "Accept:application/json" localhost:8083/
{"version":"1.0.0","commit":"cb8625948210849f"}
$ curl -H "Accept:application/json" localhost:8083/connectors/
[]

Monitor the MySQL database
Using the same terminal, we’ll use curl to submit to our Kafka Connect service a JSON request message with information about the connector we want to start. Since this command will not be in a Docker container, we need to use the IP address of our Docker host (OS X should replace localhost with their IP address):

{
  "name": "inventory-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "184054",
    "database.server.name": "dbserver1",
    "database.whitelist": "inventory",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.inventory"
  }
}

mysql> INSERT INTO customers VALUES (default, "Sarah", "Thompson", "kitt@acme.com");
mysql> INSERT INTO customers VALUES (default, "Kenneth", "Anderson", "kander@acme.com");
