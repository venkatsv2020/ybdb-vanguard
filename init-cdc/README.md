## ybdb explore the distributed sql universe

Run the following from `connector-config` shell

### create cdc stream at the source

```
yb-admin -master_addresses 127.0.0.1:7100 create_change_data_stream ysql.yugabyte

```

### create ybdb source connector

```
export STREAM_ID=$(yb-admin -master_addresses 127.0.0.1:7100 list_change_data_streams | grep stream_id | awk '{print $2}' | sed s/\"//g)

curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{
  "name": "ybsource",
  "config": {
    "tasks.max":"1",
    "connector.class": "io.debezium.connector.yugabytedb.YugabyteDBConnector",
    "topics_regex": "'${TOPIC_PREFIX}'.public.(.*)",
    "database.hostname":"'$NODE'",
    "database.master.addresses":"'$MASTERS'",
    "database.port":"5433",
    "database.user": "yugabyte",
    "database.password":"yugabyte",
    "database.dbname":"yugabyte",
    "database.server.name":"ybsource",
    "snapshot.mode":"initial",
    "database.streamid":"'$STREAM_ID'",
    "table.include.list":"public.*",
    "new.table.poll.interval.ms":"5000",
    "transforms":"Reroute",
    "transforms.Reroute.type":"io.debezium.transforms.ByLogicalTableRouter",
    "transforms.Reroute.topic.regex":"(.*)",
    "transforms.Reroute.topic.replacement":"'${TOPIC_PREFIX}'_all_events",
    "transforms.Reroute.key.field.regex":"'${TOPIC_PREFIX}'.public.(.*)",
    "transforms.Reroute.key.field.replacement":"'\$1'"
  }
}'

```

### create postgres sink connector
```
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{
  "name": "pgsink",
  "config": {
      "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
      "tasks.max": "1",
      "topics": "'${TOPIC_PREFIX}'_all_events",
      "dialect.name": "PostgreSqlDatabaseDialect",
      "connection.url": "jdbc:postgresql://localhost:5432/postgres?user='${TAR_USER}'&password='${TAR_SECRET}'",
      "auto.create": "true",
      "auto.evolve":"true",
      "insert.mode": "upsert",
      "pk.mode": "record_key",
      "delete.enabled":"true",
      "transforms": "KeyFieldExample,ReplaceField,unwrap",
      "transforms.KeyFieldExample.type": "'io.aiven.kafka.connect.transforms.ExtractTopic\$Key'",
      "transforms.KeyFieldExample.field.name": "__dbz__physicalTableIdentifier",
      "transforms.KeyFieldExample.skip.missing.or.null": "true",
      "transforms.ReplaceField.type": "'org.apache.kafka.connect.transforms.ReplaceField\$Key'",
      "transforms.ReplaceField.blacklist": "__dbz__physicalTableIdentifier",
      "transforms.unwrap.type": "io.debezium.connector.yugabytedb.transforms.YBExtractNewRecordState",
      "transforms.unwrap.drop.tombstones": "false"
   }
}'
```