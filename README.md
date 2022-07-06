1. configuration of the Debezium MySQL connector
```{
    "name": "inventory-connector",
    "config": {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "tasks.max": "1",
        "database.hostname": "mysql",
        "database.port": "3306",
        "database.user": "debezium",
        "database.password": "dbz",
        "database.server.id": "184054",
        "database.server.name": "dbserver1",                                        
        "database.whitelist": "inventory",                                          
        "database.history.kafka.bootstrap.servers": "kafka:9092",
        "database.history.kafka.topic": "schema-changes.inventory",
        "transforms": "route",                                                      
        "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",  
        "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",                     
        "transforms.route.replacement": "$3"                                        
    }
}
```
1. send to url
`curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{"name":"inventory-connector","config":{"connector.class":"io.debezium.connector.mysql.MySqlConnector","tasks.max":"1","database.hostname":"mysql","database.port":"3306","database.user":"debezium","database.password":"dbz","database.server.id":"184054","database.server.name":"dbserver1","database.whitelist":"inventory","database.history.kafka.bootstrap.servers":"kafka:9092","database.history.kafka.topic":"schema-changes.inventory","transforms":"route","transforms.route.type":"org.apache.kafka.connect.transforms.RegexRouter","transforms.route.regex":"([^.]+)\\.([^.]+)\\.([^.]+)","transforms.route.replacement":"$3"}}'`
1. Verify that inventory-connector is included in the list of connectors:
```curl -H "Accept:application/json" localhost:8083/connectors/```
3. Review the connector’s tasks:
```curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector```

4. check Kafka topic list
```./bin/kafka-topics.sh --bootstrap-server=kafka:9092 --list```
5. show kafka massage
```./bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic dbserver1.inventory.addresses --from-beginning```
6. add sink to postgres config:
```{
    "name": "jdbc-sink",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "tasks.max": "1",
        "topics": "customers",
        "connection.url": "jdbc:postgresql://postgres:5432/inventory?user=postgresuser&password=postgrespw",
        "transforms": "unwrap",                                                  
        "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
        "auto.create": "true",                                                   
        "insert.mode": "upsert",                                                 
        "pk.fields": "id",                                                       
        "pk.mode": "record_value"                                                
    }
}
```
- curl for **customer** topic
`curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{"name":"jdbc-sink","config":{"connector.class":"io.confluent.connect.jdbc.JdbcSinkConnector","tasks.max":"1","topics":"customers","connection.url":"jdbc:postgresql://postgres:5432/inventory?user=postgresuser&password=postgrespw","transforms":"unwrap","transforms.unwrap.type":"io.debezium.transforms.ExtractNewRecordState","auto.create":"true","insert.mode":"upsert","pk.fields":"id","pk.mode":"record_value"}}'`
- curl for **product** topic
`curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{"name":"jdbc-sink2","config":{"connector.class":"io.confluent.connect.jdbc.JdbcSinkConnector","tasks.max":"1","topics":"products","connection.url":"jdbc:postgresql://postgres:5432/inventory?user=postgresuser&password=postgrespw","transforms":"unwrap","transforms.unwrap.type":"io.debezium.transforms.ExtractNewRecordState","auto.create":"true","insert.mode":"upsert","pk.fields":"id","pk.mode":"record_value"}}'`

Source: 
- [Streaming data to a downstream database](https://debezium.io/blog/2017/09/25/streaming-to-another-database/)