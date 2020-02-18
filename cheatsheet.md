# Cheatsheet

## Authenticate inside mongos shell

```
db.auth("m103-admin", "m103-pass")
```

## Connect to replica set
```
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
```

## Creating new dirs 
```
mkdir -p /var/mongodb/db/{1,2,3,4,5,6,csrs1,csrs2,csrs3}
```

## Running mongod and CSRS
```
mongod -f node1.conf
mongod -f node2.conf
mongod -f node3.conf
mongod -f node4.conf
mongod -f node5.conf
mongod -f node6.conf
mongod -f csrs1.conf
mongod -f csrs2.conf
mongod -f csrs3.conf
```

## Creating user

Create `root` user:

```
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
```



## Creating simple mongod

```
mongod --port 30000 --dbpath first_mongod --logpath first_mongod/mongod.log --fork
```

## Connect to mongod

After running `mongod`, connect with following command

```
mongo --port 27001 -u m103-admin -p m103-pass
```

## Create `read/write` user:

```
db.createUser({
  user: "m103-application-user",
  pwd: "m103-application-pass",
  roles : [ {db: "applicationData", role: "readWrite"} ]
})
```