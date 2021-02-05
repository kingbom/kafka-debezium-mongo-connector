
### Create directory to store data, logs and connector plugins

```
mkdir -p $HOME/cp-kafka/zookeeper/data
mkdir -p $HOME/cp-kafka/zookeeper/log
mkdir -p $HOME/cp-kafka/kafka/data
mkdir -p $HOME/cp-kafka/connect/plugins
```

### Download Debezium MongoDB connector debezium-connector-mongodb-1.0.0.Final-plugin.tar.gz and extract zip to directory $HOME/cp-kafka/connect/plugins

```
tar -zxvf debezium-connector-mongodb-1.0.0.Final-plugin.tar.gz
mv debezium-connector-mongodb $HOME/cp-kafka/connect/plugins
```

### Download outbox event router jar from outbox-router-1.0.3.Final.jar and move it to directory outbox-event-router under $HOME/cp-kafka/connect/plugins

```
mkdir -p $HOME/cp-kafka/connect/plugins/outbox-event-router
mv outbox-router-1.0.3.Final.jar $HOME/cp-kafka/connect/plugins/outbox-event-router
```

### Build and start docker containers.

```
docker-compose up -d
```

### Create outbox event connector

```
curl POST 'http://localhost:8083/connectors' \
  '{
        "name": "mongodb-source-connector",
        "config": {
            "connector.class" : "io.debezium.connector.mongodb.MongoDbConnector",
            "tasks.max" : "1",
            "initial.sync.max.threads": "1",
            "mongodb.hosts" : "rs0/mongo1:27017",
            "mongodb.name" : "tdmongo",
            "offset.flush.interval.ms": "3000",
            "database.whitelist" : "purchase,order,warehouse,order,billing,supplier,wms, merchant",
            "collection.whitelist": "identity[.]outbox_events,product[.]outbox_events,store[.]outbox_events,warehouse[.]outbox_events,purchase[.]outbox_events,order[.]outbox_events,billing[.]outbox_events,supplier[.]outbox_events,wms[.]outbox_events, merchant[.]outbox_events",
            "transforms" : "router",
            "transforms.router.type" : "io.tdshop.connector.outbox.routing.EventRouter"
       }
    }'
```

### GET connector

```
curl POST 'http://localhost:8083/connectors/mongodb-source-connector'
```

### *if want to delete connector*

```
curl DELETE 'http://localhost:8083/connectors/mongodb-source-connector'
```

### When all containers are running, connect Kafka connecter container to MongoDB cluster network (this example use mongo-cluster)

```
docker network connect mongo-cluster connect
docker restart connect
```