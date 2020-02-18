
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

Once you runned `mongod` command, you can check if `mongod` instances really are created with `pidof mongod` command. If necessary you can kill process by pid with `kill <pid>` or `killall mongod` command

### Initiating replica set

Before creating replica set user, we need to initiate with `rs.initiate()`

After that you can check RS status with `rs.status()`. After some time you will get `PRIMARY` postfix in mongo shell

### Creating user

Before adding more `mongod` instances to RS if no user was created before we need to create `root` user.

Connect with `mongod --port <port number>`

Switch to `admin` database with `use admin`

And create root user

```
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
```

### Adding RS members

We need to reconnect with `mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"`

And then add more members with

```
rs.add("m103:27002")
rs.add("m103:27003")
```

After creating this files run CSRS with

```
mongod -f csrs1.conf
mongod -f csrs2.conf
mongod -f csrs3.conf
```

Then connect and initiate replica set with

```
mongo --port 26001
rs.initiate()
```

If not created, create user

```
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})
```

And authenticate inside mongo shell

```
db.auth("m103-admin", "m103-pass")
```

Add other members if not added

```
rs.add("192.168.103.100:26002")
rs.add("192.168.103.100:26003")
```

## Mongos

Inside `mongos` config file we specify config servers hostname and ports, so we need just run `mongos`, connect to it through `mongo` and add our RS to shard

### mongos.conf

```
sharding:
  configDB: m103-csrs/192.168.103.100:26001,192.168.103.100:26002,192,168,103.100:26003
security:
  authorization: enabled
  keyFile: /var/mongodb/pki/m103-keyfile
net:
  bindIp: localhost,192.168.103.100
  port: 26000
systemLog:
  destination: file
  path: /var/mongodb/db/mongos.log
  logAppend: true
processManagement:
  fork: true

```

Run `mongos` with `mongos -f mongos.conf`

Connect to `mongos` `mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin`

Check shard status `sh.status()`

Add our shard with `sh.addShard("m103-repl/192.168.103.100:27012")`

Restart RS if necessary

```
mongo --port 27002 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
```

## Second RS

Create folders for second shard `mkdir /var/mongodb/db/{4,5,6}` if not done so

Run second shard with

```
mongod -f node4.conf
mongod -f node5.conf
mongod -f node6.conf
```

Connect and initiate

```
mongo --port 27004
rs.initiate()
rs.status()
```

Add user and add RS members

```
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
rs.add("m103:27005")
rs.add("m103:27006")
```

Add second shard

```
sh.addShard("m103-repl-2/m103:27004")
sh.status()
```

Note: `sh.addShard("m103-repl-2/192.168.103.100:27004")` may not work, instead use above command

## Importing collection

You can do importing with

```
mongoimport --drop /dataset/products.json --port 26000 -u "m103-admin" \
-p "m103-pass" --authenticationDatabase "admin" \
--db m103 --collection products
```

### shardIdentity

In case of error

```
{
  "ok" : 0,
  "errmsg" : "Cannot accept sharding commands if sharding state has not ....
}
```

you should tackle problem manually as described [here](https://dba.stackexchange.com/questions/227797/no-shardidentity-document-mongodb)

Run status `sh.status()` and search for cluster id

```
"clusterId" : ObjectId("3b743bf3134df1f277980ard")
...
shards:
        {  "_id" : "shardname1",  "host" : "[host:port]",  "state" : 1 }
        {  "_id" : "shardname2",  "host" : "[host:port]",  "state" : 1 }
```

Then for each replica set

```
db.system.version.insert({
    "_id" : "shardIdentity",
    "clusterId" : ObjectId("3b743bf3134df1f277980ard"),
    "shardName" : "REPLICA_SHARD_NAME",
    "configsvrConnectionString" : "conf/[host]"
})
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

### no UUID returned

You can encounter the following error

```
expected primary shard to return a UUID for collection m103.products as part of 'info' field but got ...
```
