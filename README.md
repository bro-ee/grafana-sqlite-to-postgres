# Grafana SQLite to Postgres Database Migrator

[![Build Status](https://cloud.drone.io/api/badges/wbh1/grafana-sqlite-to-postgres/status.svg)](https://cloud.drone.io/wbh1/grafana-sqlite-to-postgres)
[![Go Report Card](https://goreportcard.com/badge/github.com/wbh1/grafana-sqlite-to-postgres)](https://goreportcard.com/report/github.com/wbh1/grafana-sqlite-to-postgres)

## Background
[My blog post](https://wbhegedus.me/migrating-grafanas-database-from-sqlite-to-postgres/)

I ran into the issue of Grafana logging users out because the SQLite database was locked and tokens couldn't be looked up. This is discussed in [#10727 on grafana/grafana](https://github.com/grafana/grafana/issues/10727#issuecomment-479378941). The solution is to migrate to a database that isn't the default one of SQLite.

## Prerequisites
You **must** already have an existing database in Postgres for Grafana.

Run `CREATE DATABASE grafana` in `psql` to make the database. Then, start up an instance of Grafana pointed to the new database. Grafana will automagically create all the tables that it will need. You can shut Grafana down once those tables are made. We **need** those tables to exist for the migration to work.

## Compatability
Tested on:

| OS             | SQLite Version | Postgres Version | Grafana Version |
| -------------- | -------------- | ---------------- | --------------- |
| MacOS          | 3.24.0         | 11.3             | 6.1.0+          |
| CentOS 7/RHEL7 | 3.7.17         | 11.3             | 6.1.0+          |

## Install GoLang on CentOS7
```
sudo rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
curl -s https://mirror.go-repo.io/centos/go-repo.repo | sudo tee /etc/yum.repos.d/go-repo.repo
yum install golang
```

## Usage without compilation
Do steps from section "Prerequisites"
And then:
```
git clone https://github.com/wbh1/grafana-sqlite-to-postgres.git
cd grafana-sqlite-to-postgres/cmd/grafana-migrate
go run main.go grafana.db "postgres://unix:/var/run/postgresql:5432/grafana?user=grafana"
```

## Usage
```
usage: Grafana SQLite to Postgres Migrator [<flags>] <sqlite-file> <postgres-connection-string>

A command-line application to migrate Grafana data from SQLite to Postgres.

Flags:
  --help       Show context-sensitive help (also try --help-long and --help-man).
  --dump=/tmp  Directory path where the sqlite dump should be stored.

Args:
  <sqlite-file>                 Path to SQLite file being imported.
  <postgres-connection-string>  URL-format database connection string to use in the URL format (postgres://USERNAME:PASSWORD@HOST/DATABASE).
```

## Example Command
This is the command I used to transfer my Grafana database:
```
./grafana-migrate grafana.db "postgres://postgres:PASSWORDHERE@localhost:5432/grafana?sslmode=disable"
```
Notice the `?sslmode=disable` parameter. The [pq](https://github.com/lib/pq) driver has sslmode turned on by default, so you may need to add a parameter to adjust it. You can see all the support connection string parameters [here](https://godoc.org/github.com/lib/pq#hdr-Connection_String_Parameters).

## How it works
1. Dumps SQLite database to /tmp
2. Sanitize the dump so it can be imported to Postgres
3. Import the dump to the Grafana database

## Acknowledgments
Inspiration for this program was taken from
- [haron/grafana-migrator](https://github.com/haron/grafana-migrator)
- [This blog post](https://0x63.me/migrating-grafana-from-sqlite-to-postgresql/)
