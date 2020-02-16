# MongoDB Administration

This is MongoDB Administration related guide of how to create **Replica Sets, Shards, etc.**

# Creating shard

## Creating Replica Set

Shards consists of one or multiple replica sets, CSRS (Config Server Replica Set) and **mongos** instance

Replica set is the **mongod** instances, CSRS is also **mongod** but with the special configuration option, **mongos** is used as interface for shards.

So in order to create shard we need to: first create replica set, create csrs, create and configure sharded cluster

### Replica set configuration files

Below are 3 config files for a first replica set

#### node1.conf

```
sharding:
   clusterRole: shardsvr

net:
   bindIp: 192.168.103.100,localhost
   port: 27001

storage:
   dbPath: /var/mongodb/db/1
   wiredTiger:
      engineConfig:
         cacheSizeGB: .1

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/1/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile
```

#### node2.conf

```
sharding:
  clusterRole: shardsvr
storage:
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1

net:
   bindIp: 192.168.103.100,localhost
   port: 27002

storage:
   dbPath: /var/mongodb/db/2

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/2/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile

```

#### node3.conf

```
sharding:
  clusterRole: shardsvr
storage:
  wiredTiger:
     engineConfig:
        cacheSizeGB: .1

net:
   bindIp: 192.168.103.100,localhost
   port: 27003

storage:
   dbPath: /var/mongodb/db/3

processManagement:
   fork: true

systemLog:
   destination: file
   path: /var/mongodb/db/3/mongod.log
   logAppend: true

replication:
   replSetName: m103-repl

security:
   authorization: enabled
   keyFile: /var/mongodb/pki/m103-keyfile

```

### Running replica set

You should run replica set with `mongod -f node1.conf` command for first node and similar commands for other 2

Some things should be already done though, like:

- `dbPath` already should be created

```
mkdir /var/mongodb/db/{1,2,3}
```

- keyfile should exist

```
sudo mkdir -p /var/mongodb/pki
sudo chown vagrant:vagrant -R /var/mongodb
openssl rand -base64 741 > /var/mongodb/pki/m103-keyfile
chmod 600 /var/mongodb/pki/m103-keyfile
```
