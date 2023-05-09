# ZEEK

- CPU intensive
- port independent protocal analysis: classifies traffic based of what it is and not what its riding as. metadata.
- lives on sensor  

install zeek and fsf. network plugin for kafka so it can write logs to kafka. make script for passing to fsf. fsf make log called rockoutloud to filebeat. json goes to filebeat too.

## INSTALL ZEEK

```
[root@sensor ~]# sudo yum install zeek
```
- install zeek
- now 2 packages to install
```
[root@sensor ~]# sudo yum install zeek-plugin-af_packet
```
- af_packet on nic to send to zeek
```
[root@sensor ~]# sudo yum install zeek-plugin-kafka
```
- zeek send logs over network to kafka

zeek config files are in /etc/zeek  
/etc/zeek/networks.cfg is where 

```
[root@sensor ~]# sudo vi /etc/zeek/zeekctl.cfg
```
```
     65 # Location of the log directory where log files will be archived each rotation
     66 # interval.
     67 LogDir = /data/zeek
     68 lb_custom.InterfacePrefix=af_packet::
```
- zeek control config file. manages zeek service.
- line 68 tells zeek to use af_packet

```
[root@sensor ~]# sudo vi /etc/zeek/node.cfg
```
```
      6 # This is a complete standalone configuration.  Most likely you will
      7 # only need to change the interface.
      8 #[zeek]
      9 #type=standalone
     10 #host=localhost
     11 #interface=eth0
```
```
    13 ## Below is an example clustered configuration. If you use this,
     14 ## remove the [zeek] node above.
     15 
     16 [logger]
     17 type=logger
     18 host=localhost
     19 
     20 [manager]
     21 type=manager
     22 host=localhost
     23 pin_cpus=1
     24 
     25 [proxy-1]
     26 type=proxy
     27 host=localhost
     28 
     29 [worker-1]
     30 type=worker
     31 host=localhost
     32 interface=eth1
     33 lb_method=custom
     34 lb_procs=2
     35 pin_cpus=2,3
     36 env_vars=fanout_id=77
     37 #
     38 #[worker-2]
     39 #type=worker
     40 #host=localhost
     41 #interface=eth0
```
- default is standalone. comment lines 6-11 out to be cluster configuration. uncomment 16-31. 

```
lscpu -e
```
- shows how many cpus
- what we did before is pin (assign) certain cpu's 

```
[root@sensor ~]# sudo mkdir /usr/share/zeek/site/scripts 
```
```
[root@sensor ~]# cd /usr/share/zeek/site/scripts
```

- make scripts directory

```
[root@sensor scripts]# sudo curl -LO https://repo/fileshare/zeek/extension.zeek
```
```
[root@sensor scripts]# sudo curl -LO https://repo/fileshare/zeek/extract-files.zeek
```
```
[root@sensor scripts]# sudo curl -LO https://repo/fileshare/zeek/json.zeek     
```
```
[root@sensor scripts]# sudo curl -LO https://repo/fileshare/zeek/kafka.zeek
```
```
[elastic@sensor scripts]$ sudo curl -LO https://repo/fileshare/zeek/fsf.zeek
```
```
[elastic@sensor scripts]$ sudo curl -LO https://repo/fileshare/zeek/afpacket.zeek
```

- curl zeek scripts from filesshare
- ll to verify they're all there
 
`??? scripts make zeek usable`

```
[root@sensor site]# sudo vi /usr/share/zeek/site/local.zeek
```
```
@load ./scripts/afpacket.zeek
@load ./scripts/extension.zeek

redef ignore_checksums = T;
```
- add these lines at end to point to direction of our local scripts and ignore invalid checksum

```
[root@sensor site]# sudo mkdir -p /data/zeek
```
- we configured to write logs to /data/zeek so we gotta make it 

```
sudo yum install wget

sudo mkdir ~/downloads/zeek

cd ~/downloads/zeek

sudo wget -r --no-parent -nd -l1 https://repo/fileshare/zeek 
```

##### OR

```
[root@sensor site]# sudo chown -R zeek: /data/zeek

[root@sensor site]# sudo chown -R zeek: /etc/zeek

[root@sensor site]# sudo chown -R zeek: /usr/share/zeek

[root@sensor site]# sudo chown -R zeek: /usr/bin/zeek

[root@sensor site]# sudo chown -R zeek: /usr/bin/capstats

[root@sensor site]# sudo chown -R zeek: /var/spool/zeek 
```
- change ownership of directories

```
[root@sensor site]# sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/zeek
```
```
[root@sensor site]# sudo /sbin/setcap cap_net_raw,cap_net_admin=eip /usr/bin/capstats
```
- set capabilities of zeek usr so that it can read the net traffic

```
[root@sensor site]# sudo getcap /usr/bin/zeek
/usr/bin/zeek = cap_net_admin,cap_net_raw+eip
```
```
[root@sensor site]# sudo getcap /usr/bin/capstats
/usr/bin/capstats = cap_net_admin,cap_net_raw+eip
```
- verify setting capabilities worked

```
[root@sensor site]# sudo -u zeek zeekctl deploy
```
- systemctl has limited options so we use zeekctl

```
[root@sensor scripts]# sudo -u zeek zeekctl status

Warning: ZeekControl plugin uses legacy BroControl API. Use
'import ZeekControl.plugin' instead of 'import BroControl.plugin'

Name         Type    Host             Status    Pid    Started
logger       logger  localhost        running   4763   04 May 21:12:22
manager      manager localhost        running   4810   04 May 21:12:23
proxy-1      proxy   localhost        running   4855   04 May 21:12:24
worker-1-1   worker  localhost        running   4912   04 May 21:12:26
worker-1-2   worker  localhost        running   4915   04 May 21:12:26
```
- verify everything is running

```
[elastic@sensor ~]$ cd /data

[elastic@sensor data]$ ll
total 12
drwxr-xr-x 4 stenographer stenographer 4096 May  5 07:04 stenographer
drwxr-xr-x 2 suricata     suricata     4096 May  5 08:04 suricata
drwxr-xr-x 3 zeek         zeek         4096 May  5 09:00 zeek

[elastic@sensor data]$ cd zeek

[elastic@sensor zeek]$ ll
total 4
drwxr-xr-x 2 zeek zeek 4096 May  5 18:00 2023-05-05
lrwxrwxrwx 1 zeek zeek   22 May  5 08:35 current -> /var/spool/zeek/logger

[elastic@sensor zeek]$ cd current

[elastic@sensor current]$ ll
total 24
-rw-r--r-- 1 zeek zeek  412 May  5 18:05 capture_loss.log
-rw-r--r-- 1 zeek zeek 1356 May  5 18:05 conn.log
-rw-r--r-- 1 zeek zeek 1596 May  5 18:00 dns.log
-rw-r--r-- 1 zeek zeek 1736 May  5 18:05 stats.log
-rw-r--r-- 1 zeek zeek  300 May  5 18:00 stderr.log
-rw-r--r-- 1 zeek zeek  188 May  5 08:35 stdout.log
```

```
[elastic@sensor current]$ cat conn.log
```
- navigate to the current in zeek and verify gettin traffic in conn.log

# INSTALL FSF (FILE SCANNING FRAMEWORK)

```
[elastic@sensor ~]$ sudo yum install fsf
```
- install fsf
```
[elastic@sensor ~]$ sudo vi /opt/fsf/fsf-server/conf/config.py
```
```
SCANNER_CONFIG = { 'LOG_PATH' : 'data/fsf',
                   'YARA_PATH' : '/var/lib/yara-rules/rules.yara',
                   'PID_PATH' : '/run/fsf/fsf.pid',
                   'EXPORT_PATH' : '/data/fsf/archive',
                   'TIMEOUT' : 60,
                   'MAX_DEPTH' : 10,
                   'ACTIVE_LOGGING_MODULES' : ['rockout'],
                   }

SERVER_CONFIG = { 'IP_ADDRESS' : "localhost",
                  'PORT' : 5800 }
```
- edit config file for server

```
[elastic@sensor ~]$ sudo mkdir -p /data/fsf/archive
```
```
[elastic@sensor ~]$ sudo chown -R fsf: /data/fsf
```
- create directory path we named in config file before
- change ownership

```
[elastic@sensor ~]$ sudo vi /opt/fsf/fsf-client/conf/config.py
```
```
SERVER_CONFIG = { 'IP_ADDRESS' : ['localhost',],
                  'PORT' : 5800 }

# Full path to debug file if run with --suppress-report
CLIENT_CONFIG = { 'LOG_FILE' : '/tmp/client_dbg.log' }
```
- edit config file for client

```
[elastic@sensor ~]$ sudo vi /usr/lib/systemd/system/fsf.service
```
- verify its good?

```
[elastic@sensor ~]$ sudo systemctl enable fsf --now
```
```
[elastic@sensor ~]$ sudo systemctl status fsf
```
- start and enable
- check status


### **`a good trouble shooting step to get more details on why something failed is journalctl -xeu <service>`**


---
---


## 3 SCRIPTS TO LOCAL.ZEEK?

```
[elastic@sensor ~]$ /opt/fsf/fsf-client/fsf_client.py --full interface.sh
```
- manually scanning file

```
[elastic@sensor site]$ sudo vi /usr/share/zeek/site/local.zeek
```
```
@load ./scripts/afpacket.zeek
@load ./scripts/extension.zeek
@load ./scripts/extract-files.zeek
@load ./scripts/fsf.zeek
@load ./scripts/json.zeek

redef ignore_checksums = T;
```
- added in extract-files, fsf, and json into local.zeek lines
- adding it now cus fsf wasnt installed yet and wouldve been getting called out to ultimately makeing zeek fail


> - extract-files, fsf, and json into local.zeek???? because???

```
[elastic@sensor site]$ sudo -u zeek zeekctl stop
```
```
[elastic@sensor site]$ sudo -u zeek zeekctl deploy
```
- like a restart. with zeek you have to stop and then deploy.


 ```
 [elastic@sensor site]$ sudo -u zeek zeekctl status

Warning: ZeekControl plugin uses legacy BroControl API. Use
'import ZeekControl.plugin' instead of 'import BroControl.plugin'

Name         Type    Host             Status    Pid    Started
logger       logger  localhost        running   7867   04 May 23:49:26
manager      manager localhost        running   7914   04 May 23:49:28
proxy-1      proxy   localhost        running   7959   04 May 23:49:29
worker-1-1   worker  localhost        running   8019   04 May 23:49:31
worker-1-2   worker  localhost        running   8017   04 May 23:49:31
```
- verify everything running

```
[elastic@sensor site]$ cd /data/fsf

[elastic@sensor fsf]$ ll
total 12
drwxr-xr-x 2 fsf fsf 4096 May  4 23:05 archive
-rw-r--r-- 1 fsf fsf   54 May  4 23:12 daemon.log
-rw-rw-rw- 1 fsf fsf    0 May  4 23:31 dbg.lock
-rw-rw-rw- 1 fsf fsf    0 May  4 23:31 dbg.log
-rw-rw-rw- 1 fsf fsf    0 May  4 23:31 rockout.lock
-rw-rw-rw- 1 fsf fsf 1096 May  4 23:31 rockout.log
```
- displaying ....?




