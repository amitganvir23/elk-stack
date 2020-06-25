# Install and configure Elasticsearch on AWS
Elasticsearch makes it easy to deploy, secure, operate, and scale Elasticsearch for log analytics, full text search, application monitoring, and more. 

ElasticSearch supports following use cases, 
 - Log Analytics
 - Full Text Search
 - Distributed Document Store
 - Real-time Application Monitoring
 - Clickstream Analytics

 This document deals with IaaS based implementation of ElasticSearch, If you are looking for Managed ElasticSearch Service - [AWS ElasticSearch Service FAQ](https://aws.amazon.com/elasticsearch-service/faqs/)

Follow this article on **[Youtube](https://youtu.be/7WE8AAdGSlM)**

![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/elk.png)

### Pre-Requisites
 - An EC2 Instance- [How to Create EC2 in AWS](https://www.youtube.com/watch?v=N_mP4mIqK8A&list=PLxzKY3wu0_FLaF9Xzpyd9p4zRCikkD9lE&index=11&t=0s)
 - An Security Group port to be open - [Attach SG to running EC2](https://www.youtube.com/watch?v=GlPTgGZR-j8﻿)
    - Port 9200: For REST API ( _for learning open to the internet_, usually private subnets in your VPC)
    - Port 9300: For ES Nodes to communicate ( _for learning open to the internet_, usually private subnets in your VPC) 

## Install Elasticsearch

### Step1: Pre-Requisites
The minimum required Java version is 8.
```sh
$ sudo yum -y install java-1.8.0-openjdk
$ sudo yum -y remove java-1.7.0-openjdk
$ java -version
```

Export JAVA Variables.
```sh
$ sudo cat > /etc/profile.d/java.sh << "EOF"
export JAVA_HOME=/usr/lib/jvm/jre-openjdk
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
EOF

$ source /etc/profile.d/java.sh
```

### Step2: Installing from the RPM repository
We are going to install ES in Amazon Linux 2, you should be able to adapt this procedure for say RHEL/CentOS/Fedora.
Elasticsearch requires at least Java 8.
```sh
$ sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Create a file called `elasticsearch.repo` in the `/etc/yum.repos.d/` directory. The below repo is for ES version 6.x
```sh
$ sudo cat > /etc/yum.repos.d/elasticsearch.repo << "EOF"
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

Install from RPM Repo
```sh
$ sudo yum install elasticsearch -y
```

For manual installations visit
https://www.elastic.co/downloads/elasticsearch


### Step3: Configuration of ES (Elastic Search)
Update Elasticsearch Minumum and maximum java memory. Default recomanded 1g and below replaced with 512m.
```sh
$ sudo vi /etc/elasticsearch/jvm.options
-Xms512m
-Xmx512m

:wq
```

Configure Elasticsearch and allow service through Public network or anywhere
```sh
$ sudo cat >> /etc/elasticsearch/elasticsearch.yml << "EOF"
node.name: "My First Node"
cluster.name: mycluster1
network.host: 0.0.0.0
EOF
```

### Step4: Running Elasticsearch
Use the `chkconfig` or `systemctl` command to configure Elasticsearch to start automatically when the system boots up
```sh
$ sudo chkconfig --add elasticsearch
```
Elasticsearch can be started using the `service` or `systemctl` command. Check status of the service after sometime.
```sh
$ sudo service elasticsearch start
  Starting elasticsearch:                                    [  OK  ]
$
$ sudo service elasticsearch status
```
Use the `netstat` or lsof command to verify Elasticsearch ports numbers 9200 -and 9300.
```sh
$ sudo netstat -tulpn | grep -e 9200 -e 9300
tcp6       0      0 0.0.0.0:9300          :::*                    LISTEN      12646/java
tcp6       0      0 0.0.0.0:9200          :::*                    LISTEN      12646/java
[root@ip-172-31-64-218 ~]#
```


### Step5: Lets Access to ElasticSearch
Once it is up then it will be accessable from enywhere, we can verify it and it will be see something like this:
If 9200 is allowend in your SG on AWS then we can access it from internet as well.
```sh
$ curl -X GET "http://127.0.0.1:9200?pretty"
{
  "name" : "My First Node",
  "cluster_name" : "mycluster1",
  "cluster_uuid" : "A_Sx8sMuS_WYUfdoO7-CZA",
  "version" : {
    "number" : "6.8.10",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "537cb22",
    "build_date" : "2020-05-28T14:47:19.882936Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.3",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
It is my strong recommendation that you put your servers in a load balacer. Check out how to setup [ELB here](https://www.youtube.com/watch?v=QyjDktNxdQg)

## credentials setup on ES(Elastic Search)
### Step1: Configuration of ES
Append descovery type and X-Pack plugin scurity parmaters in ES configuration. By default this plugin is alrady part of this version therefore no need to install.
```sh
$ sudo cat >> /etc/elasticsearch/elasticsearch.yml << "EOF"
discovery.type: single-node
xpack.security.enabled: true
EOF
```
### Step2: Running Elasticsearch
Elasticsearch can be started using the `service` or systemctl command. Check status of the service after sometime.
```sh
$ sudo service elasticsearch restart
  Starting elasticsearch:                                    [  OK  ]

$ sudo service elasticsearch status
```

Use the `netstat` or lsof command to verify Elasticsearch ports numbers
```sh
$ sudo netstat -tulpn | grep -e 9200 -e 9300
tcp6       0      0 0.0.0.0:9300          :::*                    LISTEN      12646/java
tcp6       0      0 0.0.0.0:9200          :::*                    LISTEN      12646/java
[root@ip-172-31-64-218 ~]#
```

### Step3: Creating credentils
There are tow option to to genrate default credentils/password to the below users, first auto and second is manualy.  It will be ontime process once x-pack configured.
apm_system
kibana
logstash_system
beats_system
remote_monitoring_user
elastic

In the below example trying to auto genrated password and it will be ontime process once x-pack configured.
```sh
$ sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]`y`

Changed password for user apm_system
PASSWORD apm_system = Vfdx19x5W3On3FMLDvGh

Changed password for user kibana
PASSWORD kibana = cybTyGeMTD93PqKCKiqw

Changed password for user logstash_system
PASSWORD logstash_system = pcYzcJM82DHoMsTDywxO

Changed password for user beats_system
PASSWORD beats_system = COXXkW4bkalcHlZbMzGl

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = YmaxNmptPsijoO2ao7Vl

Changed password for user elastic
PASSWORD elastic = IFEB688UuZGj7XEZLgRP
```

Lets Access ElasticSearch
```sh
$ curl -X GET -u elastic:IFEB688UuZGj7XEZLgRP "http://127.0.0.1:9200?pretty"
```
### Step4: New Canredentils
Create a new user, password and assgin role to the user
New user name is: bob
password for bob is 123456 (Password should min 6 characters)
To allow ES, have given role: superuser (full access)
```sh
$ sudo /usr/share/elasticsearch/bin/elasticsearch-users useradd bob -p 123456 -r superuser
```

Lets verify and listed all users created
```sh
$ sudo /usr/share/elasticsearch/bin/elasticsearch-users list
amit           : superuser
$
```
Lets Access ElasticSearch
```sh
$ curl -X GET -u bob:123456 "http://127.0.0.1:9200?pretty"
```
Incase to restet password for the user
```sh
$ sudo /usr/share/elasticsearch/bin/elasticsearch-users passwd bob
```

## Accessing the ES through GUI
Install this chrome plugin
```sh
https://github.com/mobz/elasticsearch-head
```
From your browser, poing to the public IP of your ES Cluster at port 9200
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-health-00.png)

## Next Steps
 - Pushing logs from Logstash to ElastiSearch - [Follow Here](https://github.com/miztiik/elk-stack/tree/master/Logstash) - [Youtube](https://youtu.be/YasrCKykAKo)
