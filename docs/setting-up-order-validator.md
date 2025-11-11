<h1>SETTING UP ORDER VALIDATOR</h1>

---

**Contents**:

- [Python library for Kafka](#python-library-for-kafka)
- [Validating and enriching](#validating-and-enriching)
- [Dockerfile](#dockerfile)
- [Docker Compose](#docker-compose)

---

# Python library for Kafka
[`src/app/order_validator.py`](../src/app/order_validator.py)

**Some notes on algorithmic decisions**:

*Adding schema for `enriched_orders` messages*

- The PostgreSQL sink connector was initially configured with `delete.enabled=false` and '`pk.mode=none`
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

# Validating and enriching

# Dockerfile

# Docker Compose