# MongoDB Administration

This is MongoDB Administration related guide of how to create **Replica Sets, Shards, etc.**

# Creating shard

## Creating Replica Set

Shards consists of one or multiple replica sets, CSRS (Config Server Replica Set) and **mongos** instance

Replica set is the **mongod** instances, CSRS is also **mongod** but with the special configuration option, **mongos** is used as interface for shards.

So in order to create shard we need to: first create replica set, create csrs, create and configure sharded cluster

### Replica set configuration files

Below are 