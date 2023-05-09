# Configure Environment

## Terminal  

```
lxc list
```  
- lists all containers

```
lxc start --all
```
- start all containers

```
ip a
```
- verify ip addresses 

```
ssh elastic@xxx.xxx.xxx.xxx
```
-  ssh into ip


## `if it doesnt allow you to ssh delete contents (:%d) of this vi /home/unbonto/.........`

```
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
- navigate to vim config script

``` 
:%d
```
- delete whats in the vim 


## Eth0
```
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
HOSTNAME=elastic0
NM_CONTROLLED=no
TYPE=Ethernet
IPADDR=10.81.139.30
GATEWAY=10.81.139.1
PREFIX=24
```
---
```
sudo vi /etc/hosts
```
```
%d
```
- navigate to hosts config file and delete contents

## vi /etc/hosts
```
127.0.0.1 localhost
10.81.139.10 repo
10.81.139.20 sensor
10.81.139.30 elastic0
10.81.139.31 elastic1
10.81.139.32 elastic2
10.81.139.40 pipeline0
10.81.139.41 pipeline1
10.81.139.42 pipeline2
10.81.139.50 kibana
```

---


    sudo systemctl restart network
- apply changes


    lxc list
- verify changes took affect


### `if changes not/incorrectly reflecting from student laptop lxc exec (node) /bin/bash, vi into whichever script is wrong to correct, lxc restart (node)`

---


    ssh-keygen
- generate ssh key

```
for host in sensor repo elastic{.0..2} pipeline{0..2} kibana; do ssh-copy-id $host; done  
```

- generating ssh key for each container. also makes it so that you can ssh with the hostname instead of ip address.


---


# CONFIGURE REPO


sudo yum install nginx
- install into repo

ll
- list files

sudo unzip ~/all-class-files.zip -d /usr/share/nginx
- unzips files and places it in annotated file path

sudo mv /usr/share/nginx/all-class-files /usr/share/nginx/fileshare
- move and rename

sudo mv ~/emerging.rules.tar.gz /usr/share/nginx/fileshare
- gonna be used for suricata so we move to fileshare directory for easier access

---

# IDK WHATS HAPPENIING HERE


cd /usr/share/nginx/fileshare/  
ll  
sudo vi /etc/nginx/conf.d/fileshare.conf  


...vi
```
server {
  listen 8000;
  location / {
    root /usr/share/nginx/fileshare;
    autoindex on;
    index index.html index.htm;
  }
}
```  

- listen on port 8000

sudo firewall-cmd --add-port=8000/tcp --permanent  
sudo firewall-cmd --reload  

sudo systemctl enable --now nginx  
sudo systemctl status nginx  
ss -lnt

















