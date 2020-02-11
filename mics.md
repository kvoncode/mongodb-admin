# MongoDB

## Connect to mongod

After running `mongod`, connect with following command

```
mongo --port 27001 -u m103-admin -p m103-pass
```


## Creating user (Mongo shell)

Create `root` user:

```
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles : [ "root" ]
})
```

Create `read/write` user:

```
db.createUser({
  user: "m103-application-user",
  pwd: "m103-application-pass",
  roles : [ {db: "applicationData", role: "readWrite"} ]
})
```