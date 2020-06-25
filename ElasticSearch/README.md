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

## Step 1: Install Elasticsearch

### Pre-Requisites
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

### Installing from the RPM repository
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

## Running Elasticsearch
Use the `chkconfig` command to configure Elasticsearch to start automatically when the system boots up
```sh
$ sudo chkconfig --add elasticsearch
```
Elasticsearch can be started using the `service` or systemctl command. Check status of the service after sometime.
```sh
# To Start Elasticsearch 
$ sudo service elasticsearch start
Starting elasticsearch:                                    [  OK  ]

$ sudo service elasticsearch status
```
Use the `netstat` or lsof command to verify Elasticsearch ports numbers
```sh
$ su7do netstat -tulpn|grep -e 9200 -e 9300
tcp6       0      0 0.0.0.0:9300          :::*                    LISTEN      12646/java
tcp6       0      0 0.0.0.0:9200          :::*                    LISTEN      12646/java
[root@ip-172-31-64-218 ~]#
```


#### Lets Access to ElasticSearch
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


## Accessing the ES through GUI
Install this chrome plugin
```sh
https://github.com/mobz/elasticsearch-head
```
From your browser, poing to the public IP of your ES Cluster at port 9200
![](https://raw.githubusercontent.com/miztiik/elk-stack/master/images/ELK-health-00.png)

## Next Steps
 - Pushing logs from Logstash to ElastiSearch - [Follow Here](https://github.com/miztiik/elk-stack/tree/master/Logstash) - [Youtube](https://youtu.be/YasrCKykAKo)
