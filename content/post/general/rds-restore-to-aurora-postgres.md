+++
author = "Atif Ramzan"
title = "How to restore from RDS Postgres to Aurora Postgres"
date = "2023-05-24"
description = "In this post we will take the dump of AWS RDS Postgres and restore it on the AWS Aurora Postgres"
categories = [
    "general"
]
thumbnail = "../images/aurora-postgres-image.png"
+++

In this post we will be taking the dump of existing AWS RDS Postgres and we will restore it on the Aurora Postgres Cluster. This post assumes that you have already AWS RDS Postgres and you want to move your database data to Aurora Postgres Cluster.

1. Take the database dump from the old RDS instance while going into one of the dev instances. Use to command below to dump the database. Please note that you need to add the switch   `--format=custom` otherwise, you won’t be able to restore the database on the new Postgres Aurora instance.
``` bash
pg_dump --host [database-endpoint-url] --password --username [db-username] --dbname [db-name] --file /home/ubuntu/backup-$(date +%Y-%m-%d).dump --format=custom
```
2. Make sure that if you have audit databases take the dump for that as well using the above command.
3. Copy the dump to your local using any SFTP client.
4. Upload the dump to your ec2 instance which has access to the Aurora Postgres Cluster.
5. Use the below command to restore the database. Be cautious for the below command it will wipeout the data from your database and upload the new data that you have in the dump file.
``` bash
pg_restore --verbose --clean --no-acl --no-owner -c --host [db-endpoint-url] --user [db-name] --dbname [db-name] ./backup-2023-05-02-custom.dump
```
6. If the above command gives you an error, use the below command first and then run the above command again.
``` bash
pg_restore --verbose --clean --no-acl --no-owner -c --host [db-endpoint-url] -j 2 --user [db-user] --dbname [db-name] ./backup-2023-05-02-custom.dump
```
If you notice we have used the `-j` with 2 argument, if you dump file contains more data you need to increase the worker processor for the restore command so it restore it in parallel.

7. Make sure that you have not got any errors while running the `pg_restore` command.
