---
layout: post
title:  "Elastic Search"
date:   2019-10-18 08:01:18 +0545
categories: [Ruby on Rails, Elastic search]
tags: [elastic_search]
---

# Introduction to Elastic Search

Elastic Search is a full-text search engine which can be used as NoSQL database and can be used as analytics engine.
It is schema-less, easy to scale, near real-time and provides a restful interface for different operations.

Elastic search is used as primary backend of your web application which can be added to an existing system which run through existing data source. Elastic search can be used to monitor and analysis of the existing application without affecting the behaviour of the current application.

Various UseCases of Elastic Search are:

* Web Application Search Solution
* Data Visualization and Analytics
* Log Management
* Online Database Storage
* Monitoring System
* Autocomplete and instant search


Elastic Search has following components:

* Cluster
  A cluster is a collection of one or more server/nodes which holds your entire data together and provides indexing and search capabilities across all nodes. "elasticsearch" is unique default cluster.

* Node
  A node is a single server which is a part of a cluster which stores your data, participate's in cluster's indexing and search capabilities. A node is unique name and by default Universally Unique Identifier (UUID).

* Index
  An index is a data structure which is a collection of documents having similar characteristics which is used to improve query execution time. Index are created for table primary keys, foreign keys, unique numbers, etc so that query executed 250 times faster than query without indexing.

* Type
  Type is used to store various types of data in the same index, in order to keep the total number of indices. The `_type` field is added to every document which is used for filtering the data when searching with a specific type.

* Document
	Document is the row of record for the table or collection which is a single piece of information and it can be indexed.

* Shard

	The data is shared or divided into two or multiple nodes/machines/servers in the cluster when data grows really fast and run out of space which is called as shard.
	Sharding is useful as it horizontally scale your content volume and it allows to distribute and parallelize operations across shards which increases the performance.

# Installation of ElasticSearch on Ubuntu 18.04

* You need to use sudo login
* Install the deb package from the official Elasticsearch repository

* Install apt-transport-https package that necessary to access a repository over HTTPs.

```Ruby
$ sudo apt update
$ sudo apt install apt-transport-https
```

* Install OpenJDK 8

```Ruby
sudo apt install openjdk-8-jdk
```

* Verify the java installation

```Ruby
$ java -version

# this gives output as below

openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-8u222-b10-1ubuntu1~18.04.1-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

* Import repository's GPG using the following wget command

```Ruby
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

```

* The output of above command should be OK which means the key is imported successfully and packages from this repository will be considered trusted.

* Add the Elasticsearch repository to the system by issuing:

```Ruby
sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
```

* Now update apt package list and install Elasticsearch engine by following commands:

```Ruby
sudo apt update
sudo apt install elasticsearch
```

* Start the Elasticsearch processes

```Ruby
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```

* Verify Elasticsearch is running by command

```Ruby
curl -X GET "localhost:9200/"

# it's output is as shown below

{
  "name" : "crystal-Aspire-E5-575G",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "9IFdxeCaRZmSj5c33WxEEg",
  "version" : {
    "number" : "7.4.0",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "22e1767283e61a198cb4db791ea66e3f11ab9910",
    "build_date" : "2019-09-27T08:36:48.569419Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```Ruby

* Log messages can be seen by using following command

```
sudo journalctl -u elasticsearch
```

* Some useful directories

data storage - /var/lib/elasticsearch
configuration file - /etc/elasticsearch
Java startup options configuration - /etc/default/elasticsearch

#### Remote Access Setup

Elasticsearch by default listen to localhost, so if database also in the same host, it is single node cluster and default configuration works.

Anyone can access Elasticsearch by HTTP API as Elasticsearch lacks authentication. So you need to give access to Elasticsearch server to only trusted client which is done by configuring firewall (check firewall tool UFW on ubuntu) and give access to port 9200.

First add a rule which allow incoming SSH

```Ruby
sudo ufw allow 22
```

Allow access from trusted client 

```Ruby
sudo ufw allow from 192.168.1.65 to any port 9200
# replace remote ip address with 192.168.1.65
```

Enable UFW

```Ruby
sudo ufw enable
```

Check firewall status

```Ruby
sudo ufw status

// o/p looks like this

Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
9200                       ALLOW       192.168.1.65
22 (v6)                    ALLOW       Anywhere (v6)
```

* Next edit Elasticsearch configuration allow it to listen to external connections

```Ruby
sudo vim /etc/elasticsearch/elasticsearch.yml
```

Uncomment line having `network.host`, change the value to 0.0.0.0. To make Elasticsearch listen on specified interface among multiple network interfaces on your machine you can specify interface IP address.

Restart the Elasticsearch service and now connection to Elasticsearch server from remote is ready.

```Ruby
sudo systemctl restart elasticsearch
```

{% include inarticle-adsense.html %}