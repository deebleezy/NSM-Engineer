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

# BREAK FIXES

Suricata down when you run status:  
- check suricata.yaml
- fix and restart


Zeek
- "sudo -u zeek zeekctl status" showed running
- list topics in pipeline0
    - topics were there now show if any traffic
    - run bootstrap command. if nothing in suricata-raw or fsf-raw problem would be in filebeat. zeek goes straight to kafka. take out --from-beginning to see live traffic
    - there was no live traffic
- sudo vi /usr/share/zeek/site/scripts/kafka.zeek

Filebeat
- 
- eve.json path wouldnt have grown cus there was a typo in file path for .yml file
- 

Kibana unreachable
- sudo firewall-cmd --list-al
    - fireall had port 8080 not 80
    - sudo fireall-cmd --remove-port=8080/tcp --permanent
    - sudo fireall-cmd --add-port=80/tcp --permanent
    - sudo fireall-cmd --reload


No suricata in index patterns
- pipeline0
    - check suricata-raw. aint nothin
- sensor
    - suricata status good
    - suricata.yaml
    - default log directory should be /data/suricata and restart suricata
    - eve.son is growing /data/suricata
- check filebeat


---
---


# TROUBLESHOOTING

Start with kibana status
- good so lets pull it up.
- 502 bad gateway
- check firewall, good
- kibana.yml


ZEEK
- curl something. if no zeek log then thatts a problem.

easy check for conn.log
- check size by ll in current
- curl goog.com
- check size again, should be bigger

ALL LOGS IN KIBANA ARE DELAYED
- check EACH pipeline. thats what used to feed info
- Kafka: are topics creating anything? ( --bootstrap)
- zookeper manages kafka so if kafka is good, so is zookeeper
- logstash: verify config files for em.



