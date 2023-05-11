# LOGSTASH 

lives on pipelines

```
[elastic@pipeline0 ~]$ sudo yum install logstash -y
```
```
[elastic@pipeline0 ~]$ sudo curl -LO https://repo/fileshare/logstash/logstash.tar.gz
```
```
[elastic@pipeline0 ~]$ sudo tar -zxvf logstash.tar.gz -C /etc/logstash
```
- download and unzip logstash mutations?

```
[elastic@pipeline0 ~]$ sudo chown logstash: /etc/logstash/conf.d/*
 
[elastic@pipeline0 ~]$ sudo chmod -R 744 /etc/logstash/conf.d/ruby
```
- 


```
sudo sed -i s/127.0.0.1:9092/pipeline0:9092,pipeline1:9092,pipeline2:9092/g logstash-100-input-kafka-{suricata,fsf,zeek}.conf
```
- replaces doin it individually 3 times in the next steps


`OR` 


```
[elastic@pipeline0 conf.d]$ sudo vi logstash-100-input-kafka-zeek.conf
```
```
:%s/127.0.0.1:9092/pipeline0:9092,pipeline1:9092,pipeline2:9092/g
```
- replaces the with 127.... with the pipeline stuff:
```
input {
  kafka {
    topics => ["zeek-raw"]
    add_field => { "[@metadata][stage]" => "zeek_json" }
    # Set this to one per kafka partition to scale up
    #consumer_threads => 4
    group_id => "zeek_logstash"
    bootstrap_servers => "pipeline0:9092,pipeline1:9092,pipeline2:9092"
    codec => json
    auto_offset_reset => "earliest"
    id => "input-kafka-zeek"
  }
}
```
> ALSO DO logstash-100-input-kafka-suricata.conf and logstash-100-input-kafka-fsf.conf

---

```
sudo sed -i 's/"127.0.0.1"/"elastic0", "elastic1", "elastic2"/g' logstash-9999-output-elasticsearch.conf
```
- 

`OR`

```
[elastic@pipeline0 conf.d]$ sudo vi logstash-9999-output-elasticsearch.conf
```
```
:%s/"127.0.0.1"/"elastic0", "elastic1", "elastic2"/g
```
```
sudo -u logstash /usr/share/logstash/bin/logstash -t --path.settings /etc/logstash
```
- verify everything working in logstash. should see 6 pass, 0 fail, 0 error. ran in pipeline.


> ## `^ DO ON ALL PIPELINES ^`

```
[elastic@pipeline0 conf.d]$ sudo systemctl enable logstash --now
```
- on each pipeline 

---

## VERIFY LOGSTASH GTG

```
[elastic@pipeline0 conf.d]$ sudo tail -f /var/log/logstash/logstash-plain.log
```
- should have data in there

`OR`

Navigate to Kibana  

1. Menu > Stack Management > Index Pattern > Creat
2. Stack Management
3. Index Pattern
4. Create Index Pattern

> stuff should populate


---
---


