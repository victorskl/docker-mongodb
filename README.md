# docker-mongodb

MongoDB using the [official Docker image](https://hub.docker.com/_/mongo/).

```
docker-compose build
docker-compose --project-name=dev up -d
docker ps
docker exec -it mongodb bash
mongo
> help
> show dbs
> exit
exit
```

The data is persist into `./data` - configured in `docker-compose.yml` volumes setting.

GUI tools 

- [Mongo Compass](https://www.mongodb.com/products/compass)

- [Robo 3T](https://robomongo.org/)

- http://mongodb-tools.com
