# docker-mongodb

MongoDB using the [official Docker image](https://hub.docker.com/_/mongo/).

### Standalone

```
git clone https://github.com/victorskl/docker-mongodb.git
cd docker-mongodb
mkdir -p data

docker-compose build
docker-compose -p dev up -d
docker ps
docker exec -it mongodb bash
mongo
> help
> show dbs
> exit
exit
```

The data is persist into `./data` - configured in `docker-compose.yml` volumes setting.

### GUI Tools 

- [Mongo Compass](https://www.mongodb.com/products/compass)
- [Robo 3T](https://robomongo.org/)
- http://mongodb-tools.com

---

### Replica Set

```
docker-compose -p dev down
docker-compose -p dev -f replicaset.yml -f docker-compose.yml up -d
docker exec -it mongodb bash
mongo
> rs.conf()
> rs.initiate()
> rs.status()
> rs.conf()
> exit
```

- https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/

### Going back to Standalone

- Remove all secondary hosts from replica set
- Just relaunch without `--replSet`, i.e.

```
rs0:PRIMARY> rs.conf()
rs0:PRIMARY> rs.remove('my-secondary-hostname1:27017')
rs0:PRIMARY> rs.remove('my-secondary-hostname2:27017')

docker-compose -p dev -f replicaset.yml -f docker-compose.yml down
docker-compose -p dev up -d
```

- After relaunched into the standalone mode, proceed to simply drop the [`local`](https://docs.mongodb.com/manual/reference/local-database/) database.

```
use local
db.dropDatabase()
```

- OR; 

- If you don't want to drop the `local` database, you can, at least, empty the `local.system.replset` collection. This will get back into the replica free state.

```
> use local
switched to db local
> db.system.replset.find()
{ "_id" : "rs0", "version" : 1, "protocolVersion" : NumberLong(1), "members" : [ { "_id" : 0, "host" : "79a9cc330a08:27017", "arbiterOnly" : false, "buildIndexes" : true, "hidden" : false, "priority" : 1, "tags" : {  }, "slaveDelay" : NumberLong(0), "votes" : 1 } ], "settings" : { "chainingAllowed" : true, "heartbeatIntervalMillis" : 2000, "heartbeatTimeoutSecs" : 10, "electionTimeoutMillis" : 10000, "catchUpTimeoutMillis" : -1, "catchUpTakeoverDelayMillis" : 30000, "getLastErrorModes" : {  }, "getLastErrorDefaults" : { "w" : 1, "wtimeout" : 0 }, "replicaSetId" : ObjectId("5ab48c1fbba4ee5aae46d9bc") } }
> db.system.replset.remove({})
WriteResult({ "nRemoved" : 1 })
> db.system.replset.find()
>
```

- Perhaps; you might also want to check the [oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/) as well. Because `oplog` is the [capped-collection](https://docs.mongodb.com/manual/reference/glossary/#term-capped-collection), you can not remove records from it. To reset the `oplog`, just drop it. Next time, when you re-initiate the replica set, the `oplog` will be created again.

```
> use local
> show collections
> db.oplog.rs.find()
> rs.printReplicationInfo()
> db.oplog.rs.drop()
true
```

### Config File

If you like to run with config file:

```
cp -v mongod.conf.sample mongod.conf
docker-compose -p dev -f mongod-with-conf.yml -f docker-compose.yml up -d
```

Or; you could just modify it in `docker-compose.yml`, instead of overriding.