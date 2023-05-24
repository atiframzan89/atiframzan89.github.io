+++
author = "Atif Ramzan"
title = "How to restore from RDS Postgres to Aurora Postgres"
date = "2023-05-24"
description = "In this post we will take the dump of AWS RDS Postgres and restore it on the AWS Aurora Postgres"
categories = [
    "general"
]
+++

In this post we will be taking the dump of existing AWS RDS Postgres and we will restore it on the Aurora Postgres Cluster. This post assumes that you have already AWS RDS Postgres and you want to move your database data to Aurora Postgres Cluster.

1. Take the database dump from the old RDS instance while going into one of the dev instances. Use to command below to dump the database. Please note that you need to add the switch `--format=custom` otherwise, you won’t be able to restore the database on the new Postgres Aurora instance.
2. 