# docker-mongodb

MongoDB using the [official Docker image](https://hub.docker.com/_/mongo/).

## Local Development

```
git clone https://github.com/victorskl/docker-mongodb.git
cd docker-mongodb
mkdir -p data

docker compose up -d
docker compose ps

docker exec -it mongo bash
mongosh
test> help
test> show dbs
test> use admin
admin> show collections
admin> show users
admin> show roles
admin> exit
exit
```

The data is persist into `./data` - configured in `compose.yml` volumes setting.

Reset like so:
```
docker compose down
rm -rf ./data
mkdir -p data
```

## GUI

_desktop_

- [Mongo Compass](https://www.mongodb.com/products/tools/compass)

```
brew install --cask mongodb-compass
```

_browser_

```
docker compose -f express.yml -f compose.yml up -d
docker compose ps
```

Open in browser like so:
```
open -a "Google Chrome" http://localhost:8081/
```

```
docker exec -it mongo bash
mongosh -u root -p example
test> show dbs
test> use admin
admin> show collections
admin> exit
exit
```

```
docker compose -f express.yml -f compose.yml down
```

## Replica Set

```
docker compose -f replicaset.yml -f compose.yml up -d
docker compose ps
```

- https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/

```
docker exec -it mongo bash
mongosh
> rs.conf()
> rs.initiate()
> rs.status()
> rs.conf()
```

### Convert Replica Set back to Standalone

- Remove all secondary hosts from replica set
- Just relaunch without `--replSet`, i.e.

```
rs0:PRIMARY> rs.conf()
rs0:PRIMARY> rs.remove('my-secondary-hostname1:27017')
rs0:PRIMARY> rs.remove('my-secondary-hostname2:27017')

docker compose -f replicaset.yml -f compose.yml down
```

Restart without replica set like so:
```
docker compose up -d
```

- After relaunched into the standalone mode, simply drop the [`local`](https://docs.mongodb.com/manual/reference/local-database/) database.

```
use local
db.dropDatabase()
```

- If you do not want to drop the `local` database, you can, at least, empty the `local.system.replset` collection. This will get back into the replica free state.

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

- Perhaps; you might also want to check the [oplog](https://docs.mongodb.com/manual/core/replica-set-oplog/) as well. Because `oplog` is the [capped-collection](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-capped-collection), you can not remove records from it. To reset the `oplog`, just drop it. Next time, when you re-initiate the replica set, the `oplog` will be created again.

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
docker compose -f conf.yml -f compose.yml up -d
docker compose ps
docker compose -f conf.yml -f compose.yml down
```
