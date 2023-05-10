# How to verify Zeek, FSF, and Suricata logs in Kafka

## This will create the Topic
sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic <topicname>

## This will list all topics in the Kafka cluster
sudo /usr/share/kafka/bin/kafka-topics.sh --list --zookeeper pipeline0:2181

## This will list information about the Topic
sudo /usr/share/kafka/bin/kafka-topics.sh --describe --zookeeper pipeline0:2181 --topic <topicname>

## This will print to stdout ALL messages produced to a Kafka topic from the beginning of the topic.
sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic <topicname>--from-beginning

## This will print to stdout ONLY NEW messages to a Kafka topic as they are produced
sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic <topicname>

---
---

