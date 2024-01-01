---
layout: post
title:  "Backup and Restore PostgreSQL Database"
date:   2018-08-27 09:23:58 +0545
categories: [PostgreSQL, Backup Restore]
tags: [postgres, backup_restore]
---

### Backup Database
##### pg_dump is the PostgreSQL utility to backup the database.

##### To backup a single database, run below command in command line interface as superuser.

```
~ sudo pg_dump -U postgres -h localhost name-of-database > name-of-backup-file
```

### Restore Database

##### Backup file created can be useful to restore your system.
##### To restore it is essential to create a empty database and then you can restore database using following command:
```
psql new_Database_name < path_to_backup_file
```

##### Here is the full script to restore the single database

```
sudo su - postgres
postgres@usr-Aspire-E5-575G:~$ psql
postgres=# CREATE DATABASE new_database_name TEMPLATE template0;
postgres@usr-Aspire-E5-575G:~$ psql new_database_name < /home/siv/name-of-backup-file
```

{% include inarticle-adsense.html %}