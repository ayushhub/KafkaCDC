We will use a distributed platform that turns our existing databases into event streams, so applications can see and respond immediately to each row-level change in the databases. Our platform (Debezium) is built on top of Apache Kafka and provides Kafka Connect compatible connectors that monitor specific database management systems. This records the history of data changes in Kafka logs, from where the application consumes them. This makes it possible for the application to easily consume all of the events correctly and completely. Even if the application stops (or crashes), upon restart it will start consuming the events where it left off so it misses nothing.

In Production: 
Production environments, require running multiple instances of each service to provide the performance, reliability, replication, and fault tolerance. 
This can be done with a platform like OpenShift and Kubernetes that manages multiple Docker containers running on multiple hosts and machines on dedicated hardware.

Caveat: You cannot remove a row that is referenced by a foreign key.
