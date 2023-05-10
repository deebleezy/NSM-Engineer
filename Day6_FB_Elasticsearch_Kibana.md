# FILEBEAT
- lightweight log shipper. 
- ships logs to kafka
- install on on sensor


```
[elastic@sensor ~]$ sudo yum install filebeat -y
```
- install filebeat

```
[elastic@sensor ~]$ sudo mv /etc/filebeat/filebeat{.yml,.yml.bk}
```
- rename current basic yml and make it a backup filebeat config file just inscase we fudge ours up.
- we have a config we're gonna curl down so just in case 

```
[elastic@sensor ~]$ cd /etc/filebeat/
```
```
[elastic@sensor filebeat]$ sudo curl -LO https://repo/fileshare/filebeat/filebeat.yml

```
- 
```
[elastic@sensor filebeat]$ sudo vi /etc/filebeat/filebeat.yml
```
```
     33 output.kafka:
     34   hosts: ["pipeline:9092","pipeline1:9092","pipeline2:9092"]
```
- pointing it to pipelines cus thats where kafka lives. not localhost.

```
[elastic@pipeline0 ~]$ sudo /usr/share/kafka/bin/kafka-topics.sh --list --zookeeper pipeline0:2181

__consumer_offsets
zeek-raw
```
```
[elastic@pipeline0 ~]$ sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic fsf-raw

Created topic fsf-raw.
```
```
[elastic@pipeline0 ~]$ sudo /usr/share/kafka/bin/kafka-topics.sh --create --zookeeper pipeline0:2181 --replication-factor 3 --partitions 3 --topic suricata-raw

Created topic suricata-raw.
```
```
[elastic@pipeline0 ~]$ sudo /usr/share/kafka/bin/kafka-topics.sh --bootstrap-server pipeline0:9092 --list

__consumer_offsets
fsf-raw
suricata-raw
zeek-raw
```
- lists out current topics. should only be zeek-raw right now.
- create fsf-raw
- create suricata-raw
- verify topic list

```
[elastic@sensor filebeat]$ sudo systemctl start filebeat
```
```
[elastic@sensor filebeat]$ ^start^status

sudo systemctl status filebeat
● filebeat.service - Filebeat sends log files to Logstash or directly to Elasticsearch.
   Loaded: loaded (/usr/lib/systemd/system/filebeat.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-05-08 18:49:57 UTC; 35s ago
     Docs: https://www.elastic.co/products/beats/filebeat
 Main PID: 33561 (filebeat)
   CGroup: /system.slice/filebeat.service
           └─33561 /usr/share/filebeat/bin/filebeat -environment systemd -c /...

May 08 18:49:57 sensor systemd[1]: Started Filebeat sends log files to Logs.....
Hint: Some lines were ellipsized, use -l to show in full.
```
- start filebeat
- verify running


```
sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic fsf-raw --from-beginning
```
```
sudo /usr/share/kafka/bin/kafka-console-consumer.sh --bootstrap-server pipeline0:9092 --topic suricata-raw --from-beginning
```
- verify messages are added to fsf-raw
- verify messages are added to suricata-raw

---
---

# ELASTICSEARCH

```
[elastic@elastic0 ~]$ sudo yum install elasticsearch -y
```
```
[elastic@elastic0 ~]$ sudo mv /etc/elasticsearch/elasticsearch{.yml,.yml.bk}
```
- install and backup default settings

```
[elastic@elastic0 ~]$ sudo curl -LO https://repo/fileshare/elasticsearch/elasticsearch.yml
```
```
[[elastic@elastic2 ~]$ sudo vi /etc/elasticsearch/elasticsearch.yml
```

```
cluster.name:  nsm-cluster
node.name:  es-node-0
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
http.port:9200
network.host: _local:ipv4_
discovery.type: single-node
 ```
> __THIS IS FOR SINGLE NODE__

OR

```
cluster.name: nsm-cluster
node.name: es-node-0
path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch
bootstrap.memory_lock: true
network.host: _site:ipv4_
http.port: 9200
discovery.seed_hosts: ["elastic0", "elastic1", "elastic2"]
cluster.initial_master_nodes: ["es-node-0", "es-node-1", "es-node-2"]
```

> __THIS IS FOR CLUSTER.__

> __UPDATE NODE NAME.__

```
[elastic@elastic0 ~]$ sudo chmod 640 /etc/elasticsearch/elasticsearch.yml
```
- change elastic config ownership to ????

```
[elastic@elastic0 ~]$ sudo mkdir /usr/lib/systemd/system/elasticsearch.service.d
```
```
[elastic@elastic0 ~]$ sudo chmod 755 /usr/lib/systemd/system/elasticsearch.service.d
```
- make service directory
- change ownership to ???

```
[elastic@elastic0 ~]$ sudo vi /usr/lib/systemd/system/elasticsearch.service.d/override.conf
```
```
[Service]
LimitMEMLOCK=infinity
```

```
[elastic@elastic0 ~]$ sudo chmod 644 /usr/lib/systemd/system/elasticsearch.service.d/override.conf 
```
```
[elastic@elastic0 ~]$ sudo systemctl daemon-reload
```

- create override config in service directory
- stop memory swapping with elasticsearch. not good to do that.
- change ownership to ???
- reload somethiiing ???


---


### JVM STUFF ?

create jvm options.  
good rule of thumb, half of ram so other systems have available ram but not over 31.

```
[elastic@elastic0 ~]$ sudo vi /etc/elasticsearch/jvm.options.d/jvm_override.conf
```
```
-Xms2g
-Xmx2g
```
- 

```
[elastic@elastic0 ~]$ sudo mkdir -p /data/elasticsearch

[elastic@elastic0 ~]$ sudo chown elasticsearch: /data/elasticsearch/

[elastic@elastic0 ~]$ sudo chmod 755 /data/elasticsearch/
```
-
-
-

```
[elastic@elastic0 ~]$ sudo firewall-cmd --add-port={9200,9300}/tcp --permanent

success
```
```
[elastic@elastic0 ~]$ sudo firewall-cmd --reload

success
```
- 

```
[elastic@elastic0 system]$ sudo systemctl start elasticsearch
```
```
[elastic@elastic0 system]$ sudo systemctl status elasticsearch

● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; disabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/elasticsearch.service.d
           └─override.conf
   Active: active (running) since Mon 2023-05-08 21:10:41 UTC; 2min 37s ago
     Docs: https://www.elastic.co
 Main PID: 2640 (java)
   CGroup: /system.slice/elasticsearch.service
           ├─2640 /usr/share/elasticsearch/jdk/bin/java -Xshare:auto -Des.networkaddress.cache.ttl=60 ...
           └─2835 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

```
- start elastic and verufy running


```
[elastic@elastic0 ~]$ curl elastic0:9200/_cat/nodes?v

ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.81.139.31           51          56   0    1.87    1.71     1.45 dilmrt    *      es-node-1
10.81.139.30           18          56   0    1.87    1.71     1.45 dilmrt    -      es-node-0
10.81.139.32           35          55   0    1.87    1.71     1.45 dilmrt    -      es-node-2
```
- verify cluster is set up
- add ?v to show header so you know what the info is


---
---

# KIBANA

```
[elastic@kibana ~]$ sudo yum install kibana -y

[elastic@kibana ~]$ sudo mv /etc/kibana/kibana{.yml,.yml.bk}
 
[elastic@kibana ~]$ sudo vi /etc/kibana/kibana.yml
```
```
server.port: 5601
server.host: localhost
server.name: kibana
elasticsearch.hosts: ["http://elastic0:9200","http://elastic1:9200","http://elastic2:9200"]
```

- install kibana
- move and rename default yml to be backup
- create and configue new yml file as current yml

```
[elastic@kibana ~]$ sudo yum install nginx -y

[elastic@kibana ~]$ sudo vi /etc/nginx/conf.d/kibana.conf
```
```
server {
  listen 80;
  server_name kibana;
  proxy_max_temp_file_size 0;

  location / {
    proxy_pass http://127.0.0.1:5601/;

    proxy_redirect off;
    proxy_buffering off;

    proxy_http_version 1.1;
    proxy_set_header Connection "Keep-Alive";
    proxy_set_header Proxy-Connection "Keep-Alive";

  }

}
```
- install nginx
- create and edit kibana config file in nginx

```
[elastic@kibana ~]$ sudo vi /etc/nginx/nginx.conf
```
```
     39 #        listen       80;
     40 #        listen       [::]:80;
     41 #        server_name  _;
```
- edit nginx config file to stop listening on port 80

```
[elastic@kibana ~]$ sudo firewall-cmd --add-port=80/tcp --permanent

success
```
```
[elastic@kibana ~]$ sudo firewall-cmd --reload 

success
```
- reverse proxy through nginx. we ignored port 80 in the config file but allowing 80 to ride through nginx. thats why we dont have to put port # in browser to navigate.
- refresh firewall


```
[elastic@kibana ~]$ sudo systemctl enable nginx --now

Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
```
```
[elastic@kibana ~]$ sudo systemctl enable kibana --now

Created symlink from /etc/systemd/system/multi-user.target.wants/kibana.service to /etc/systemd/system/kibana.service.
```
```
[elastic@kibana ~]$ sudo systemctl status nginx
```
```
[elastic@kibana ~]$ sudo systemctl status kibana
```
- enable and start nginx and kibana
- verify both are active and running

### `you should now be able to navigate to http://kibana in browser`


---

```
[elastic@kibana ~]$ ll

total 4
drwxrwxr-x 2 elastic elastic 4096 May  5 01:52 archive
```
```
[elastic@kibana ~]$ sudo curl -LO https://repo/fileshare/kibana/ecskibana.tar.gz


  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3370  100  3370    0     0  19626      0 --:--:-- --:--:-- --:--:-- 19707
```
```
[elastic@kibana ~]$ ll

total 8
drwxrwxr-x 2 elastic elastic 4096 May  5 01:52 archive
-rw-r--r-- 1 root    root    3370 May  9 00:50 ecskibana.tar.gz
```
- curl down the **???** into kibana

```
[elastic@kibana ~]$ tar -zxvf ecskibana.tar.gz

ecskibana/
ecskibana/export-index-templates.sh
ecskibana/default.json
ecskibana/ecs_suricata.json
ecskibana/ecs-mapping.json
ecskibana/failure_index.json
ecskibana/import-index-templates.sh
ecskibana/ecs_zeek.json
ecskibana/no_replicas.json
```
- uncompress the folder filled with ????

```
[elastic@kibana ~]$ sudo yum install jq -y
```
- install the jq **???**

```
[elastic@kibana ecskibana]$ sudo ./import-index-templates.sh http://elastic0:9200

Installing index mapping template: default version 1
Installing index mapping template: ecs_mapping version 1
Installing index mapping template: ecs_suricata version 1
Installing index mapping template: ecs_zeek version 1
Installing index mapping template: failure_index version 1
Installing index mapping template: no_replicas version 1
OK: 0
Changed: 6
Failed: 0
```
- pushing over indexs from pipeline0














blah

blah  

blah











