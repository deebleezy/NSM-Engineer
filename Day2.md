# CREATE LOCAL REPOSITORY

yum
- search, install, update instances

---

## PATCH LOCALLY VIA REPO SERVER

> `sync repo in local directory. created directory for packages. yum cant see it, has to see metadata`

```
ssh repo
```

```
sudo yum install yum-utils  
```
- ssh into repo node
- install yum utilities. add -y to skip the "yes" prommpts

```
sudo reposync -l --repoid=extras --download_path=/repo/local-extras
```
- creating another local repo, syncing with the centos

```
sudo yum install createrepo
```  
```
cd /repo
```
```
sudo createrepo local-extras
```
 
```
sudo vi /etc/nginx/conf.d/packages.conf
```
- install createrepo
- create configuration file for packages to install. 
```
 server {
  listen 8008;
  location / {
    root /repo;
    autoindex on;
    index index.html index.htm;
  }
}
```
- specify port to listen to and where packages live.  
```
sudo firewall-cmd --add-port=8008/tcp --permanent
```
```
sudo firewall-cmd --reload
```
```
sudo firewall-cmd --list-all
```
```
ss -lnt
```
- allow that port to come through
- reload/refresh firewall to have changes take affect
- verify ports listed

- verify socket list
```
sudo systemctl restart nginx
```
- apply changes to nginx
```
^restart^status
```
- verify everything is active and running

 > `should now be able to navigate to repo:8008 via webpage`


---
---


 # PULL UPDATES FROM REPO SERVER ( http://repo:8008 )
 ## SKIP THIS FOR REBUILD

---

```
ssh sensor
```
- ssh into node 
```
sudo yum list zeek
```
- 
```
mkdir ~/archive
```
-  make directory in home drive
```
ll /etc/yum.repos.d
```
- listing out whats in the yum directory where it currently is
```
sudo mv /etc/yum.repos.d/* ~/archive
```
- move the contents of EVERYTHING (anotated bt the *) to the home archive directory
```
sudo curl -LO http://repo:8000/local.repo
```
- curl (download) this file 
```
[elastic@sensor yum.repos.d]$ sudo vi /etc/yum.repos.d/local.repo
```
- cd into yum.repos.d and update hostname 
```
[local-base]
name=local-base
baseurl=http://repo:8008/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=http://repo:8008/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=http://repo:8008/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=http://repo:8008/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=http://repo:8008/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=http://repo:8008/local-updates/
enabled=1
gpgcheck=0
```
- update hostname to repo:8008

```
[elastic@sensor yum.repos.d]$ sudo yum makecache
```
- making a local cache. changing repo config 

---
---

# SECURING REPO AND FILESHARE
 ## SKIP THIS FOR REBUILD

---

```
[elastic@repo ~]$ mkdir ~/certs 
[elastic@repo ~]$ cd ~/certs
[elastic@repo certs]$ openssl genrsa -des3 -out localCA.key 2048
Generating RSA private key, 2048 bit long modulus
.........+++
............................................+++
e is 65537 (0x10001)
Enter pass phrase for localCA.key:
Verifying - Enter pass phrase for localCA.key:
[elastic@repo certs]$
```
- make a cert directory using des3 encrytption named localCA.key. bit leangth 2048.


```
[elastic@repo certs]$ openssl req -x509 -new -key localCA.key -sha -days 1095 -out -localCA.crt
Enter pass phrase for localCA.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:CA
Locality Name (eg, city) [Default City]:Mountain View
Organization Name (eg, company) [Default Company Ltd]:Elastic
Organizational Unit Name (eg, section) []:Security
Common Name (eg, your name or your server's hostname) []:Instructor
Email Address []:
[elastic@repo certs]$
```
- using key for ssl  for nodes. using key named localCA.key for 3 years and gonna call it localCA.crt
```
[elastic@repo certs]$ openssl genrsa -out repo.key 2048
```
- create repo private key


```
sudo vi ~/certs/repo.ext
```
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = repo
IP.1 = 10.81.139.10
```
- create vim to be able to browse to fileshare. subject outname to specify repo and ip so that we can type in name in web browser instead of ip.
```
[elastic@repo certs]$ openssl x509 -req -in repo.csr -CA -localCA.crt -CAkey localCA.key -CAcreateserial -out repo.crt -days 365 -sha256 -extfile repo.ext
Signature ok
subject=/C=US/ST=CA/L=Mountain View/O=Elastic/OU=Security/CN=Repo
Getting CA Private Key
Enter pass phrase for localCA.key:
```
```
[elastic@repo certs]$ sudo mv repo.crt /etc/nginx
```
```
[elastic@repo certs]$ sudo mv repo.key /etc/nginx
```
```
[elastic@repo certs]$ ll /etc/nginx
total 84
drwxr-xr-x 2 root    root    4096 May  2 18:59 conf.d
drwxr-xr-x 2 root    root    4096 Nov 10 16:58 default.d
-rw-r--r-- 1 root    root    1077 Nov 10 16:58 fastcgi.conf
-rw-r--r-- 1 root    root    1077 Nov 10 16:58 fastcgi.conf.default
.
.
.
```
- create private key (.key) and move it and digital certificate (.csr) to nginx 
```
[elastic@repo certs]$ cd /etc/nginx/conf.d      
```
```
[elastic@repo conf.d]$ sudo curl -LO http://repo:repo:8000/nginx/proxy.conf
```
- 

```  
server {
  listen 127.0.0.1:8000;
  location / {
    root /usr/share/nginx/fileshare;
    autoindex on;
    index index.html index.htm;
  }
}
```
- added in loopback ip address in vim for 
- 

- 
```
[elastic@repo conf.d]$ sudo firewall-cmd --add-port=
```
```
[elastic@repo conf.d]$ sudo firewall-cmd --add-port={80,443}/tcp --permanent
 
success
```
```
[elastic@repo conf.d]$ sudo firewall-cmd --remove-port={8000,8008}/tcp --permanent

success
```
```
[elastic@repo conf.d]$ sudo firewall-cmd --reload

success
```
```
[elastic@repo conf.d]$ sudo firewall-cmd --list-all

public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: dhcpv6-client ssh
  ports: 80/tcp 443/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```
```
[elastic@repo conf.d]$ sudo systemctl restart nginx
```
```
[elastic@repo conf.d]$ ^restart^status
```
> `stop listening to public ports and listen to private ports`

---
---

# PULL UPDATES FROM REPO SERVER ( http://repo/packages )


```
sudo vi /etc/yum.repos.d/local.repo
```
```
ubuntu@ip-172-31-17-37:~$ ssh elastic0
```
```
[elastic@elastic0 ~]$ mkdir ~/archive
``` 
```
[elastic@elastic0 ~]$ sudo mv /etc/yum.repos.d/* ~/archive
```
```
[elastic@elastic0 ~]$ ll
total 52
-rw-r--r-- 1 elastic elastic 1664 May  2 23:22 CentOS-Base.repo
-rw-r--r-- 1 elastic elastic 1309 May  2 23:22 CentOS-CR.repo
-rw-r--r-- 1 elastic elastic  649 May  2 23:22 CentOS-Debuginfo.repo
.
.
.
```
```
[elastic@elastic0 ~]$ sudo vi /etc/yum.repos.d/local.repo
```
```
[local-base]
name=local-base
baseurl=http://repo/packages/local-base/
enabled=1
gpgcheck=0

[local-rocknsm-2.5]
name=local-rocknsm-2.5
baseurl=http://repo/packages/local-rocknsm-2.5/
enabled=1
gpgcheck=0

[local-elasticsearch-7.x]
name=local-elasticsearch-7.x
baseurl=http://repo/packages/local-elastic-7.x/
enabled=1
gpgcheck=0

[local-epel]
name=local-epel
baseurl=http://repo/packages/local-epel/
enabled=1
gpgcheck=0

[local-extras]
name=local-extras
baseurl=http://repo/packages/local-extras/
enabled=1
gpgcheck=0

[local-updates]
name=local-updates
baseurl=http://repo/packages/local-updates/
enabled=1
gpgcheck=0
```
```
ll /etc/yum.repos.d
```
-   1. ssh into node  
    2. make "archive" directory in home (~) directory
    3. move /etc/yum.d/* to ~/archive
    4. make local repo file
    5. paste same config file info from earlier used for redirecting patch pull
    6. ensures that local.repo is the only file in /etc/yum.repos.d

> ### **`do this on every node`**


### `by default all nodes are pulling updates from online. the changes above made it so that it pulls from the repository server`

---
---



# CERTIFICATES

```
for host in elastic{0..2} pipeline{0..2} kibana sensor; do sudo scp ~/certs/localCA.crt elastic@$host:~/localCA.crt && ssh -t elastic@$host 'sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/ && sudo update-ca-trust'; done
```

##### `OPTIONAL. REPLACES NEXT 2 SECTIONS`
-  for loop to copy the local cert from the repo's home directory to all the containers and update the ca-trust.


## COPY CA CERTIFICATE TO ALL THE CONTAINERS 

```
[elastic@repo certs]$ for host in sensor elastic{0..2} pipeline{0..2} kibana; do scp ~/certs/localCA.crt elastic@$host:/home/elastic/localCA.crt ; done
```
- this will cause all container to reflect localCA.crt in home directory

---

## MOVE CA CERT TO UPDATE CA-TRUST

```
sudo mv ~/localCA.crt /etc/pki/ca-trust/source/anchors/localCA.crt
```
```
sudo update-ca-trust
```
```
sudo yum makecache fast
```
-   1. authenticate to repo
    2. updating the cert
    3. create a local cache
> ### **`do this on every node`**
```
ubuntu@ip-172-31-17-37:~$ sudo scp elastic@repo:/home/elastic/certs/localCA.crt ~/localCA.crt
```
- copy localCA cert from to home folder of student laptop












