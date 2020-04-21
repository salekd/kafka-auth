# SASL/PLAIN Authentication

```
cp server.properties ~/confluent-5.4.1/etc/kafka/server.properties

confluent local start kafka

kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test

kafka-topics --list --bootstrap-server localhost:9092
kafka-topics --list --bootstrap-server localhost:9093 --command-config client.properties_admin

echo 42 | kafka-console-producer --broker-list localhost:9092 --topic test
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_alice
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_bob
echo 42 | kafka-console-producer --broker-list localhost:9093 --topic test --producer.config client.properties_admin

kafka-topics --delete --bootstrap-server localhost:9092 --topic test

confluent local stop kafka

confluent local destroy
```
