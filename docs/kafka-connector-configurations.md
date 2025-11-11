> [Previous: Setting Up](./setting-up.md) :: [Next: Installation Verification](./installation-verification.md)

<h1>KAFKA CONNECTOR CONFIGURATIONS</h1>

---

# HTTP source connector configuration
**JSON**:

```json
{
  "name": "http-source",
  "config": {
    "connector.class": "io.confluent.connect.http.HttpSourceConnector",
    "topic.name.pattern": "orders",
    "url": "http://host.docker.internal:3000/orders",
    "http.offset.mode": "SIMPLE_INCREMENTING",
    "http.initial.offset": "1",
    "request.interval.ms": "86400000",
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter.schemas.enable": "false",
    "value.converter.schemas.enable": "false",
    "confluent.topic.bootstrap.servers": "kafka:29092",
    "confluent.topic.replication.factor": "1",
    "confluent.topic.security.protocol": "PLAINTEXT"
  }
}
```

Explanation for certain parameters:

- `"url": "http://host.docker.internal:3000/orders"`:
    - URL to which to make requests; the response is ingested as data
    - `host.docker.internal` is a special DNS name:
        - Used within Docker containers to refer to the host machine where Docker is running
        - Allows containers to access services or applications running directly on the host <br> *Rather than within another container*
        - Since the connector to be configured lives in a container, this DNS name is needed <br> *To make requests to the JSON server running on the host*
- `"http.offset.mode": "SIMPLE_INCREMENTING"`:
    - Indicates how offsets are computed and how requests are generated
    - If set to `SIMPLE_INCREMENTING`, the ${offset} used to generate requests is: <br> *Previous offset (or http.initial.offset) + Number of records in the response*
    - In this mode, `http.initial.offset` needs to be set to an integer value
- `"request.interval.ms": "86400000"`:
    - Time interval between 2 requests to the URL
    - Prevents the source connector from ingesting the same data repeatedly
    - The given interval is in milliseconds; 86400000 ms = 24 hours
- `"key.converter": "org.apache.kafka.connect.json.JsonConverter"`:
    - Converts ingested data keys into JSON format
    - Ensures message keys are appropriately parsed for the `orders` Kafka topic
-  `"value.converter": "org.apache.kafka.connect.json.JsonConverter"`:
    - Converts ingested data values into JSON format
    - Ensures message values are appropriately parsed for the `orders` Kafka topic
- `"key.converter.schemas.enable": "false"` and `"value.converter.schemas.enable": "false"`:
    - Ensure no schemas are expected by the connector
    - Hence, it shall simply ingest raw data as is
-  `"confluent.topic.bootstrap.servers": "kafka:29092"`:
      -  Indicates the Docker bridge address and port on which the Kafka broker is exposed
      -  Ensures communications are routed to the appropriate broker
 
> **References**:
>
> - [*Configuration Reference for HTTP Source Connector for Confluent Platform*, **docs.confluent.io**](https://docs.confluent.io/kafka-connectors/http-source/current/configuration.html)
> - [*HTTP Source Connector for Confluent Cloud*, **docs.confluent.io/cloud/current**](https://docs.confluent.io/cloud/current/connectors/cc-http-source.html) (for example usage)
> - [*Kafka Connect Deep Dive – Converters and Serialization Explained*, **www.confluent.io/blog**](https://www.confluent.io/blog/kafka-connect-deep-dive-converters-serialization-explained/)

# PostreSQL sink connector configuration
```json
{
  "name": "postgres-sink",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "topics": "enriched_orders",
    "connection.url": "jdbc:postgresql://host.docker.internal:5432/kafkadb",
    "connection.user": "kafkauser",
    "connection.password": "kafkapass",
    
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "true",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    
    "auto.create": "true",
    "insert.mode": "insert",
    "delete.enabled": "true",
    "table.name.format": "enriched_orders",
    "pk.mode": "record_key",
    "pk.fields": "order_id"
  }
}
```

Explanation for certain parameters:

- `"value.converter.schemas.enable": "true"`:
    - Ensures
- `"auto.create": "true"`:
    - Automatically creates a PostgreSQL database table if not existing
    - Avoids issues when the required table has been dropped
- `"insert.mode": "insert`:
    - Specifies that data must be inserted into the table
    - Essentially specifies data entry via insert queries
- `"pk.mode": "record_key"` and `"pk.fields": "order_id"`:
    - Primary key settings
    - Note that the primary key values have to be unique in the data source

Observations influencing configuration:

- This connector was initially configured with `delete.enabled=false` and '`pk.mode=none`
- These configurations require:
    - Records with a non-null Struct value and non-null Struct schema
    - As an aside, this means the schema has to be defined for each Kafka message in desired topics
- However, `delete.enabled=true` only works for `pk.mode=record_key`
- The above mode requires the configuration of `pk.fields`
- Now, note that:
    - Messages without schema are read as `class java.util.HashMap`
    - However, to identify the `pk.field`, we need `org.apache.kafka.connect.data.Struct`
    - Hence, schema was added for messages in `enriched_orders` via [`order-validator` service](./setting-up-order-validator.md)
- Either way, it proved necessary to define the schema for Kafka messages in `enriched_orders`

> **References**:
>
> - [*PostgreSQL Sink (JDBC) Connector for Confluent Cloud*, **docs.confluent.io/cloud/current**](https://docs.confluent.io/cloud/current/connectors/cc-postgresql-sink.html)