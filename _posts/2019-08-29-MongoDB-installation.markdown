---
layout: post
title:  "How to install MongoDB on Ubuntu 18.04?"
date:   2019-08-29 09:23:58 +0545
categories: [MongoDB, Installation]
tags: [mongodb, installation]
---

## Install MongoDB Community Edition on Unbuntu 18.04 (Bionic)

Import the MongoDB public GPG Key from https://www.mongodb.org/static/pgp/server-4.2.asc
```
wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
```

Create the list file /etc/apt/sources.list.d/mongodb-org-4.2.list for your ubuntu version
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
```

Reload local package database
```
sudo apt-get update
```

Install MongoDB packages
```
sudo apt-get install -y mongodb-org
```

Optional. Although you can specify any available version of MongoDB, apt-get will upgrade the packages when a newer version becomes available. To prevent unintended upgrades, you can pin the package at the currently installed version:
```
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

Directories are at

```
/var/lib/mongodb
/var/log/mongodb
```

Configuration files are at
```
/etc/mongod.conf
```

Start MongoDB
```
sudo service mongod start
```

To verify MongoDB is started successfully, check the following content on /var/log/mongodb/mongod.log
```
[initandlisten] waiting for connections on port 27017
```

Process

```
sudo service mongod stop
sudo service mongod restart
```

Use MongoDB using command shell
```
mongo
```

Uninstall MongoDB steps

```
sudo service mongod stop
sudo apt-get purge mongodb-org* # remove mongoDB packages
# remove data directories
sudo rm -r /var/log/mongodb 
sudo rm -r /var/lib/mongodb
```

{% include inarticle-adsense.html %}