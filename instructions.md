# Instructions

Initial goal is to create sharding collection. Collection data will exist on multiple shards. Each shard is a replica set. 

The standard way is to create 2 shards (RS), 1 CSRS (Config Server Replica set), and mongos instances

mongos is `mongos` executable, each (RS/shard) consist of multiple `mongod` executable instances

Running 2 shards and 1 CSRS is very similar, the only difference is configuration files of `mongod`

## Running replica set/shard

Here is example of how to run replica set/shard


## Handling errors

Most hard to debug and most common errors is messing up config files, log files and folder. Deleting mongodb folders and starting from scratch should help