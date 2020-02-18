# Instructions

Initial goal is to create sharding collection. Collection data will exist on multiple shards. Each shard is a replica set. 

The standard way is to create 2 shards (RS), 1 CSRS (Config Server Replica set), and mongos instances

mongos is `mongos` executable, each (RS/shard) consist of multiple `mongod` executable instances

Running 2 shards and 1 CSRS is very similar, the only difference is configuration files of `mongod`

## Running replica set/shard

Here is example of how to run replica set/shard

Run 
```
mongod -f node1.conf
mongod -f node1.conf
mongod -f node1.conf
```

Check with
```
pidof mongod
```

### Initiating replica set

```
rs.initiate()
rs.status()
```

### Create user
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

## Handling errors

Most hard to debug and most common errors is messing up config files, log files and folder. Deleting mongodb folders and starting from scratch should help