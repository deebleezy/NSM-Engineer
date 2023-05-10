# CONFIGURE CAPTURE INTERFACE

2 nics. 1 for capture and 1 for management

```
[elastic@sensor ~]$ sudo yum install ethtool
```
- install eth tool to manage nics. now using local-base instead of CentOS
- `eth0 management intercface. eth1 capture interface.`

```
[elastic@sensor ~]$ sudo ethtool -k eth1
```
- show us features of nic and do it on eth1
- wanna disable nic offloading to relieve cpu 

```
[elastic@sensor ~]$ sudo curl -LO https://repo/fileshare/interface.sh
```
```
[elastic@sensor ~]$ sudo chmod +x interface.sh
```
```
[elastic@sensor ~]$ sudo ./interface.sh eth1

turning off offloading on eth1
Cannot change large-receive-offload
Cannot change RX network flow hashing options: Operation not supported
Cannot change RX network flow hashing options: Operation not supported
Cannot get device coalesce settings: Operation not supported
Cannot get device coalesce settings: Operation not supported
Cannot get device ring settings: Operation not supported
```
```
[elastic@sensor ~]$ sudo ethtool -k eth1
```
- put interface scipt in home directory???
- modify file to be an execute scripts
- run executable
- verify checksums turned off


### **`if you dont do this you'll see your own zeek logs`**


```
[elastic@sensor ~]$ sudo vi /sbin/ifup-local
```
```
#!/bin/bash
if [[ "$1" == "eth1" ]]
then
for i in rx tx sg tso ufo gso gro lro rxvlan txvlan
do
/usr/sbin/ethtool -K $1 $i off
done
/usr/sbin/ethtool -N $1 rx-flow-hash udp4 sdfn
/usr/sbin/ethtool -N $1 rx-flow-hash udp6 sdfn
/usr/sbin/ethtool -n $1 rx-flow-hash udp6
/usr/sbin/ethtool -n $1 rx-flow-hash udp4
/usr/sbin/ethtool -C $1 rx-usecs 10
/usr/sbin/ethtool -C $1 adaptive-rx off
/usr/sbin/ethtool -G $1 rx 4096

/usr/sbin/ip link set dev $1 promisc on

fi
```
- script will add variable to capture int and running through features to turn off. also setting in promiscuous mode.

```
[elastic@sensor ~]$ sudo chmod +x /sbin/ifup-local
```
- makes script an excutable so that we can run it

```
[elastic@sensor ~]$ sudo vi /etc/sysconfig/network-scripts/ifup
```
```
if [ -x /sbin/ifup-local ]; then
/sbin/ifup-local ${DEVICE}
fi
```
- add these lines to ifup-local script so that it will run that executable

```
[elastic@sensor ~]$ sudo vi /etc/sysconfig/network-scripts/ifup-eth1
```
```
DEVICE=eth1
BOOTPROTO=none
ONBOOT=yes
NM_CONTROLLED=no
TYPE=Ethernet
```
- network script for eth1 configuration. this is the bare minimum to make it a useable interface

```
[elastic@sensor ~]$ sudo systemctl restart network
```
```
[elastic@sensor ~]$ sudo ethtool -k eth1
```
- apply changes and verify everything stayed off

```
[elastic@sensor ~]$ sudo tcpdump -nn -i eth1
```
```
[elastic@sensor ~]$ sudo tcpdump -nn -i eth1 '!port 22'
```

- test capture int with tcp dump. dont want to resolve ip -nn, specify int -i eth1
- create filter to see traffic other than ssh

---
---

# STENOGRAPHER

```
[elastic@sensor ~]$ sudo yum install stenographer
```
```
[elastic@sensor ~]$ sudo yum install which
```
- install stenographer
- install which?
```
[elastic@sensor ~]$ cd /etc/stenographer
```
```
[elastic@sensor stenographer]$ sudo vi config
```

```
{
  "Threads": [
    { "PacketsDirectory": "/data/stenographer/packets"
    , "IndexDirectory": "/data/stenographer/index"
    , "MaxDirectoryFiles": 30000
    , "DiskFreePercentage": 30
    }
  ]
  , "StenotypePath": "/usr/bin/stenotype"
  , "Interface": "eth1"
  , "Port": 1234
  , "Host": "127.0.0.1"
  , "Flags": []
  , "CertPath": "/etc/stenographer/certs"
}
```
- 

```
[elastic@sensor stenographer]$ sudo mkdir -p /data/stenographer/{packets,index}
```
- creating these directorys in steno config file. it has to know where its gonna write to (/packets) and where its gonna index (/index)

```
[elastic@sensor stenographer]$ sudo chown -R stenographer:stenographer /data/stenographer
```
- change permissions and group (ownership) from root root to stenographer stenographer

```
[elastic@sensor stenographer]$ sudo stenokeys.sh stenographer stenographer
Generating CA state
Generating key/cert for 'client'
Generating key/cert for 'server'
```
```
[elastic@sensor stenographer]$ ll /etc/stenographer/certs
total 36
drwx------ 2 root         root         4096 May  3 19:56 ca
-r--r--r-- 1 root         root         1866 May  3 19:56 ca_cert.pem
-r-------- 1 root         root         3247 May  3 19:56 ca_key.pem
-r--r--r-- 1 root         root         6460 May  3 19:56 client_127.0.0.1_client_cert.pem
-r--r----- 1 root         stenographer 3243 May  3 19:56 client_127.0.0.1_client_key.pem
lrwxrwxrwx 1 root         root           32 May  3 19:56 client_cert.pem -> client_127.0.0.1_client_cert.pem
lrwxrwxrwx 1 root         root           31 May  3 19:56 client_key.pem -> client_127.0.0.1_client_key.pem
-r--r--r-- 1 root         root         6441 May  3 19:56 server_127.0.0.1_cert.pem
-r-------- 1 stenographer root         3247 May  3 19:56 server_127.0.0.1_key.pem
lrwxrwxrwx 1 root         root           25 May  3 19:56 server_cert.pem -> server_127.0.0.1_cert.pem
lrwxrwxrwx 1 root         root           24 May  3 19:56 server_key.pem -> server_127.0.0.1_key.pem

```
- we changed ownership so we create certificates so that it can be used securely

```
[elastic@sensor stenographer]$ sudo systemctl enable stenographer --now
Created symlink from /etc/systemd/system/multi-user.target.wants/stenographer.service to /usr/lib/systemd/system/stenographer.service.
```
```
[elastic@sensor stenographer]$ sudo systemctl status stenographer
● stenographer.service - packet capture to disk
   Loaded: loaded (/usr/lib/systemd/system/stenographer.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-03 20:02:59 UTC; 16s ago
 Main PID: 1923 (stenographer)
   CGroup: /system.slice/stenographer.service
           ├─1923 /usr/bin/stenographer
           └─1930 /usr/bin/stenotype --threads=1 --iface=eth1 --dir=/tmp/stenographer30979...

May 03 20:02:59 sensor systemd[1]: Started packet capture to disk.

```
- start and enable stenographer. "enable" makes it so that it will persist upon a reboot.
- verify status

```
[elastic@sensor ~]$ ping 8.8.8.8
```
```
[elastic@sensor ~]$ sudo stenoread 'host 8.8.8.8'
20:08:06.780590 IP sensor > dns.google: ICMP echo request, id 1995, seq 11, length 64
20:08:06.791580 IP dns.google > sensor: ICMP echo reply, id 1995, seq 11, length 64
.
.
.
```
```
[elastic@sensor ~]$ sudo stenoread 'host 8.8.8.8' -nn
Running stenographer query 'host 8.8.8.8', piping to 'tcpdump -nn'
reading from file /dev/stdin, link-type EN10MB (Ethernet)
20:07:56.769583 IP 10.81.139.20 > 8.8.8.8: ICMP echo request, id 1995, seq 1, length 64
20:07:56.780500 IP 8.8.8.8 > 10.81.139.20: ICMP echo reply, id 1995, seq 1, length 64
.
.
.
```
- generate traffic
- see traffic
- dont want it to resolve the hostname so we can see the ip address

```
[elastic@sensor ~]$ watch ls -all /data/stenographer/packets
```
- verify steno is getting traffic

```
[elastic@sensor packets]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       243G   77G  166G  32% /
none            492K  4.0K  488K   1% /dev
devtmpfs         16G     0   16G   0% /dev/fuse
tmpfs           100K     0  100K   0% /dev/lxd
tmpfs           100K     0  100K   0% /dev/.lxd-mounts
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  8.5M   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
tmpfs           763M     0  763M   0% /run/user/0
tmpfs           763M     0  763M   0% /run/user/1000
```
- 
- set /dev/root ???? to 30 so when it gets to 70% it will start to delete (first in first out)

---
---

# CONFIGURE SURICATA

```
[elastic@sensor ~]$ sudo yum install suricata
```

` most of our config files live in /etc `

```
[elastic@sensor ~]$ ll /etc/suricata
ls: cannot open directory /etc/suricata: Permission denied

[elastic@sensor ~]$ sudo -s

[root@sensor elastic]# ll /etc/suricata
total 92
-rw-r----- 1 suricata suricata  4258 Dec 18  2019 classification.config
-rw-r----- 1 suricata suricata  1375 Dec 18  2019 reference.config
drwxr-x--- 2 suricata suricata  4096 May  3 20:36 rules
-rw-r----- 1 suricata suricata 70174 Dec 18  2019 suricata.yaml
-rw-r----- 1 suricata suricata  1644 Dec 18  2019 threshold.config
```
- 

```
[root@sensor elastic]# vi /etc/suricata/suricata.yaml
```
```
  56 default-log-dir: /data/suricata/
```
```
     58 # global stats configuration
     59 stats:
     60   enabled: no

```
```
  76       enabled: no
```
```
    402   # Stats.log contains data from various counters of the Suricata engine        .
    403   - stats:
    404       enabled: no
    405       filename: stats.log

```
```
    553   # Define your logging outputs.  If none are defined, or they are all
    554   # disabled you will get the default - console output.
    555   outputs:
    556   - console:
    557       enabled: no

```
```
    578 # Linux high speed capture support
    579 af-packet:
    580   - interface: eth1  
    581     # Number of receive threads. "auto" uses the number of cores
    582     #threads: 3

```
```
    980 # Run suricata as user and group.
    981 #run-as:
    982 #  user: suricata
    983 #  group: suricata

```
```
   1498     # Profiling can be disabled here, but it will still have a
   1499     # performance impact if compiled in.
   1500     enabled: no

```
```
   1514   # per keyword profiling
   1515   keywords:
   1516     enabled: no
```
```
   1520   prefilter:
   1521     enabled: no
```
```
   1525   # per rulegroup profiling
   1526   rulegroups:
   1527     enabled: no

```
```
   1534     # Profiling can be disabled here, but it will still have a
   1535     # performance impact if compiled in.
   1536     enabled: no

```
- 
---

```
[root@sensor elastic]# sudo vi /etc/sysconfig/suricata
```
```
OPTIONS="--af-packet=eth1 --user suricata --group suricata"
```
- 
--- 

```
[root@sensor elastic]# vi /etc/suricata/suricata.yaml
```
```
   1432 # Suricata is multi-threaded. Here the threading can be influenced.
   1433 threading:
   1434   set-cpu-affinity: yes
   1435   # Tune cpu affinity of threads. Each family of threads can be bound
   1436   # on specific CPUs.
```
```
  1452         cpu: [ "0-2" ]
  1453         mode: "exclusive"
  1454         # Use explicitely 3 threads and don't compute number by using
  1455         # detect-thread-ratio variable:
  1456         # threads: 3
  1457         prio:
  1458           low: [ 0 ]
  1459           medium: [ 1 ]
  1460           high: [ 2 ]
  1461           default: "high"
 ```
- 

```
[root@sensor elastic]# sudo suricata-update add-source emergingthreats https://repo/fileshare/emerging.rules.tar.gz
```
- suricata update to point the the emerging threats fileshare while offline
```
[root@sensor elastic]# sudo suricata-update
```
- update our local rules
```
[root@sensor elastic]# sudo mkdir -p /data/suricata
```
```
[root@sensor elastic]# sudo chown -R suricata:suricata /data/suricata
```
```
[root@sensor elastic]# ll /data
total 8
drwxr-xr-x 4 stenographer stenographer 4096 May  3 19:50 stenographer
drwxr-xr-x 2 suricata     suricata     4096 May  3 21:23 suricata
```
```
[root@sensor elastic]# sudo systemctl enable suricata --now
Created symlink from /etc/systemd/system/multi-user.target.wants/suricata.service to /usr/lib/systemd/system/suricata.service.
```
- make a suricata directory in the data directory
- root group and ownership by default so change to suricata group and owner
- verify
- start and enable

---
---


# BLEW IT UP
start from scratch, minus repo.



sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0






