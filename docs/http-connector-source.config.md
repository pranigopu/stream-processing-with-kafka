 ``` json
{
  "name": "HttpSourceConnector",
  "config": {
    "connector.class": "io.confluent.connect.http.HttpSourceConnector",
    "topic.name.pattern": "orders",
    "url": "http://data-source:3000/orders",
    "tasks.max": "1",
    "http.offset.mode": "SIMPLE_INCREMENTING",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "confluent.topic.bootstrap.servers": "kafka:29092",
    "confluent.license": "",
    "confluent.topic.replication.factor": "1"
  }
}
```