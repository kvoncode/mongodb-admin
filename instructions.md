# Instructions

- Initial goal is to create sharding collection
- Collection data will exist on multiple shards
- Each shard is a replica set. 

- The standard way is to create 2 **shards/RSs (Replica Sets), 1 CSRS (Config Server Replica set), and mongos** instances

- **mongos** is `mongos` executable, each **RS/shard** consist of multiple `mongod` executable instances

- Running each shard and CSRS is very similar, the only difference is configuration files of `mongod`

## Running replica set/shard

### Prerequisites

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

### Initiating replica set
Here is example of how to run replica set/shard

Run `mongod`
```
mongod -f node1.conf
mongod -f node1.conf
mongod -f node1.conf
```

Check with
```
pidof mongod
```

Kill if necessary
```
kill <pid>
killall mongod
```

Initiate replica set
```
rs.initiate()
rs.status()
```

Create user
```
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
```

### Adding RS members

Reconnect to RS
```
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
```

Add RS members
```
rs.add("m103:27002")
rs.add("m103:27003")
```

Check RS status
```
rs.status()
```

The same procedure is necessary for second shard and CSRS

## Mongos

After running CSRS, run `mongos`

```
mongos -f mongos.conf
```

Connect to mongos
```
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
```

Check shard status
```
sh.status()
```

Add shard
```
sh.addShard("m103-repl/192.168.103.100:27001")
```

Restart RS if necessary
```
mongo --port 27002 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
```

## Importing collection

Import with
```
mongoimport --drop /dataset/products.json --port 26000 -u "m103-admin" \
-p "m103-pass" --authenticationDatabase "admin" \
--db m103 --collection products
```

## Enabling sharding

Enable sharding with

```
sh.enableSharding("m103")
```

Creating index

```
db.products.createIndex({"<shard_key>": 1})
```

Choosing shardkey

```
db.adminCommand( { shardCollection: "m103.products", key: { <shard_key>: 1 } } )
```

## Handling errors

Most hard to debug and most common errors is messing up config files, log files and folder. Deleting mongodb folders and starting from scratch should help