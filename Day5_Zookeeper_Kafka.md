# CONFIGURE PIPELINES FOR KAFKA & ZOOKEEPER

- groups info into topics like
- using zookeeper that keeps track of nodes

```
[elastic@pipeline0 ~]$ sudo yum install kafka zookeeper
```
- install kafka and zookeeper
- add -y at the end to skip them asking for you to put it in


---
---


##  ZOOKEEPER

```
[elastic@pipeline0 ~]$ sudo mkdir -p /data/zookeeper   
```
```
[elastic@pipeline0 ~]$ sudo chown -R zookeeper: /data/zookeeper/
```
- make data directory
- change ownership

```
[elastic@pipeline0 ~]$ sudo vi /etc/zookeeper/zoo.cfg
```

```
# where zookeeper will store its data
 dataDir=/data/zookeeper

 # what port should clients like kafka connect on
 clientPort=2181

 # how many clients should be allowed to connect, 0 = unlimited
 maxClientCnxns=0

 # list of zookeeper nodes to make up the cluster
 # First port is how followers and leaders communicate
 # Second port is used during the election process to determine a leader
 server.1=pipeline0:2888:3888
 server.2=pipeline1:2888:3888
 server.3=pipeline2:2888:3888

 # more than one zookeeper node will have a unique server id.
 # Ex.) server.1, server.2, etc..

 # milliseconds in which zookeeper should consider a single tick
 tickTime=2000

 # amount of ticks a follow has to connect and sync with the leader
 initLimit=5

 # amount of ticks a follower has to sync with a leader before being dropped
 syncLimit=2
```

- creating cluster with server to pipeline config and then what they can talk on

```
[elastic@pipeline0 ~]$ sudo touch /data/zookeeper/myid
```
```
[elastic@pipeline0 ~]$ sudo chown zookeeper: /data/zookeeper/myid
```
- make file /myid
- change ownership

```
[elastic@pipeline0 ~]$ echo '1' | sudo tee /data/zookeeper/myid 
1
```
```
[elastic@pipeline0 ~]$ cat /data/zookeeper/myid 
1
```

- this makes it so it will easily identify it as 1
- verify 

```
[elastic@pipeline0 ~]$ sudo firewall-cmd --add-port={2181,2888,3888}/tcp --permanent
success
```
```
[elastic@pipeline0 ~]$ sudo firewall-cmd --reload
success
```

- 2181 is zookeeper defualt port. 2888 and 3888 is how they talk. specified in zookeeper cfg file.

> ### do this on every pipeline THEN enable and start each one 

```
[elastic@pipeline2 ~]$ sudo systemctl enable zookeeper --now
```
```
[elastic@pipeline2 ~]$ sudo systemctl status zookeeper
```
- start and enable
- verify active and running

```
ubuntu@ip-172-31-17-37:~$ for host in pipeline{0..2}; do (echo "stats" | nc $host 2181 -q 2); done

Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /10.81.139.1:55596[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 4
Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /10.81.139.1:46720[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 4
Proposal sizes last/min/max: -1/-1/-1
Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /10.81.139.1:58538[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: follower
Node count: 4
```
- this for loop in the local terminal will verify the stats of the cluster.
- should have only 1 leader and the rest followers

---
---

## CONFIGURE KAFKA

makin a cluster

```
[elastic@pipeline0 ~]$ sudo mkdir -p /data/kafka
```
```
[elastic@pipeline0 ~]$ sudo chown kafka: /data/kafka
```
- 

```
[elastic@pipeline0 ~]$ sudo cp /etc/kafka/server{.properties,.properties.bk}
```
- copy properties directory to properties.bk (backup) just incase we fudge up the next few steps

```
[elastic@pipeline0 /]$ sudo vi /etc/kafka/server.properties
```
```
:%d
```
```
# The unique id of this broker should be different for each kafka node. Good practice is to match the kafka broker id to the zookeeper server id.
broker.id=0

# the port in wich kafka should use to communicate with other kafka clients
port=9092
# the hostname or IP address in which the server listens on
listeners=PLAINTEXT://pipeline0:9092

# hostname that will be advertised to producers and consumers
advertised.listeners=PLAINTEXT://pipeline0:9092

# number of threads used to send network responses
num.network.threads=3

# number of threads used to make I/O requests
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# where kafka should write its data to
log.dirs=/data/kafka

# how many partitions and replicas should be generated for topics that are created by other software
num.partitions=3
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2
default.replication.factor = 3
min.insync.replicas = 2

# how many threads should be used for shutdown and start up
num.recovery.threads.per.data.dir=3

# how long should we retain logs in kafka
log.retention.hours=12
log.retention.bytes=90000000000

# max size of a single log file
log.segment.bytes=1073741824

# frequency in miliseconds to check if a log needs to be deleted
log.retention.check.interval.ms=300000
log.cleaner.enable=false

# will not allow a node to be elected leader if it is not in sync with other nodes. Prevents possible missing messages
unclean.leader.election.enable=false

# automatically create topics from external software
auto.create.topics.enable=false


# how to connect kafka to zookeeper
zookeeper.connect=pipeline0:2181,pipeline1:2181,pipeline2:2181
zookeeper.connection.timeout.ms=30000
```
- do every pipeline and update lines 2, 7, and 10

```
sudo firewall-cmd --add-port=9092/tcp --permanent
```
```
[elastic@pipeline0 ~]$ sudo firewall-cmd --reload
```
```
[elastic@pipeline0 /]$ sudo systemctl start kafka           
```
```
[elastic@pipeline0 /]$ sudo systemctl status kafka
```
- add kafka port
- allow it
- start and verify

> do on every pipeline  
> ---
>  __dont enable kafka before zookeeper__ 

### CREATING & DELETING TOPIC
```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --create --topic test --partitions 3 --replication-factor 3
```
```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --list
```
```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --describe --topic test
```
```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --delete --topic test
```
only on pipeline0
- Creates The Topic
- Lists The Topic
- Lists Out The Topic & Shows Replication/Partition
- deletes the topic in kafka we specified (test) so if you run list command again for test it wont be there 

## CREATE AND REPLICATE ZEEK-RAW TOPIC

```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic zeek-raw

Created topic zeek-raw.
```
```
[elastic@pipeline0 /]$ sudo /usr/share/kafka/bin/kafka-topics.sh --describe --zookeeper pipeline0:2181 --topic zeek-raw

Topic:zeek-raw	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: zeek-raw	Partition: 0	Leader: 0	Replicas: 0,1,2	Isr: 0,1,2
	Topic: zeek-raw	Partition: 1	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
	Topic: zeek-raw	Partition: 2	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
```
- create 
- verify replication

```
[elastic@sensor scripts]$ sudo vi /usr/share/zeek/site/local.zeek
```
```
@load ./scripts/kafka.zeek
```
- add to end of script

```
[elastic@sensor current]$ sudo vi /usr/share/zeek/site/scripts/kafka.zeek 
```
```
@load Apache/Kafka/logs-to-kafka
redef Kafka::topic_name = "zeek-raw";
redef Kafka::json_timestamps = JSON::TS_ISO8601;
redef Kafka::tag_json = F;
redef Kafka::send_all_active_logs = T;
redef Kafka::kafka_conf = table(
    ["metadata.broker.list"] = "pipeline0:9092,pipeline1:9092,pipeline2:9092");

```
```
[elastic@sensor ~]$ sudo -u zeek zeekctl deploy
```
---
```
ubuntu@ip-172-31-17-37:~$ ping 8.8.8.8
```
```
sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic zeek-raw
```


ping from local laptop to generate traffic. once it stops (^C)  you should see traffic in all pipelines running that command.



