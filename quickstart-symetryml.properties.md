# Kafka Configuration Properties

The following properties can be set in the Kafka connector configuration file. Other Kafka-specific configuration properties can be added, of course.

## Required Kafka Properties

`connector.class`
- com.sml.kafka.connector.SymetryKafkaSinkConnector

`topics`
- Comma-separated list of topics. However, we normally use just 1 topic with multiple partitions.

`key.converter`
- org.apache.kafka.connect.storage.StringConverter

`value.converter`
- io.confluent.connect.avro.AvroConverter or org.apache.kafka.connect.json.JsonConverter

`value.converter.schema.registry.url`
- For when `value.converter` is AvroConverter.

`value.converter.schemas.enable`
- For when `value.converter` is JsonConverter. In this case, set to true.

`tasks.max`
- The number of SymetryKafkaSinkTasks that are created for the connector. Normally, we make this the same as the number of partitions in a topic.

## Required SymetryML Properties

These properties define the name of a SymetryML project to update and how to access the service that hosts the project.

##### Single SymetryML Project

When there is just one project for the connector to update, set the following list of properties:

`sml.url`
- The URL of the service that hosts the SymetryML project.

`sml.project`
- The name of the SymetryML project to be updated.

`sml.customer.id`
- The SymetryML customer identifier for the project.

`sml.customer.secret.key`
- The SymetryML customer secret key for the project.

##### Multiple SymetryML Projects

Alternatively, the connector can update multiple SymetryML projects in multiple services. In this case, there is a 1-1 mapping between SymetryKafkaSinkTask and project, so the enumeration of the properties must equal the number of SymetryKafkaSinkTasks:

`sml.url.x`
- The URL of the service that hosts SymetryML project x.

`sml.project.x`
- The name of the SymetryML project x to be updated.

`sml.customer.id.x`
- The SymetryML customer identifier for project x.

`sml.customer.secret.key.x`
- The SymetryML customer secret key for project x.

For example, for tasks.max = 2, the list of properties may look like this:

```
"sml.url.1": "http://centos1:8080/symetry/rest",
"sml.project.1": "kafka_prj_1",
"sml.customer.id.1": "cust1",
"sml.customer.secret.key.1": "AAAAAAAAAAAAAAAAAAAAAA==",
"sml.url.2": "http://centos2:8080/symetry/rest",
"sml.project.2": "kafka_prj_2",
"sml.customer.id.2": "cust2",
"sml.customer.secret.key.2": "BBBBBBBBBBBBBBBBBBBBBB==",
```


## Optional SymetryML Properties

`sml.attribute.types`
- The comma-separated list of attribute types (each type is specified by its one-character alias) that SymetryML recognizes. Each type is mapped to its corresponding schema field in order. If omitted, the connector guesses the attribute type for each field.
- default: guess

`sml.dataframe.max.chunk.size`
- The maximum row size of each DataFrame that is sent to be learned on the SymetryML project.
- default: 500

`http.client.idle.timeout.ms`
- The idle connection timeout in the internal REST utility.
- default: 55000

`http.client.response.buffer.size`
- The size of the response buffer in the internal REST utility.
- default: 2048000

`logging.is.verbose`
- Set the logging level in the internal REST utility.
- default: false

`logging.is.signature.included`
- Include the MD5 signature in logging of requests in the internal REST utility.
- default: false

`logging.is.url.body.truncated`
- Cap the number characters of the body of each request that are logged in the internal REST utility.
- default: true

`sml.reconnect.backoff.max.ms`
- The maximum period in ms allowed between tries to send a DataFrame to the SymetryML service. This value specifies the longest possible delay between tries. See our implementation of the graceful backoff algorithm in `sml.reconnect.backoff.ms` for how it is used.
- default: 10000

`sml.reconnect.backoff.ms`
- The base period in ms between tries to send a DataFrame to the SymetryML service. The graceful backoff algorithm derives the actual delay from this value.
- default: 5000
- Here is the Java code that implements the delay between tries in the algorithm:

```
// Backoff
double bDbl = Math.pow(2, inTries); // inTries is limited by 'max tries' configured by sml.backoff.max.retries
int b = (int) bDbl;
int baseWait = mRetryDelayMs * b; // mRetryDelayMs configured by sml.reconnect.backoff.ms
// Random jitter.
double j = mRandom.nextDouble();
int waitMs = (int) (baseWait * j);
if (waitMs > mRetryDelayMaxMs) {  // mRetryDelayMaxMs configured by sml.reconnect.backoff.max.ms
    waitMs = mRetryDelayMaxMs;
}
...
Thread.sleep(waitMs);
```

`sml.backoff.max.retries`
- The maximum number of times to retry sending a DataFrame to the SymetryML service before giving up.
- default: 5
- The total number of tries is this number plus one, e.g. 5 + 1 = 6 total tries.

`sml.project.must.preexist`
- The boolean indication that the SymetryML project must exist on the service before the connector starts. If true, then a ConnectException will be thrown if the project does not exist. If false, then the Connector will create the project on the service.
- default: false

`sml.create.project.type`
- The type of SymetryML project to create on the service when needed.
- default: cpu

`sml.create.project.params`
- The property parameters of the SymetryML project to create on the service when needed.
- The default is the empty string, i.e. no characters.

`sml.create.project.persist`
- The boolean indication that SymetryML project to create on the service when needed will persist itself.
- default: false


# Sample configuration File
```
{
  "name": "sml_housing_sink",
  "config": {
    "connector.class": "com.sml.kafka.connector.SymetryKafkaSinkConnector",
    "group.id": "connect-group-1",
    "tasks.max": "1",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "topics": "sml_housing",
    "consumer.max.poll.records": 500,
    "sml.attribute.types": "C,C,C,B,C,C,C,C,C,C,C,C,C,C",
    "sml.dataframe.max.chunk.size": "100",
    "sml.url": "http://centosvm:8080/symetry/rest",
    "sml.project": "housing_prj",
    "sml.customer.id": "user",
    "sml.customer.secret.key": "secret.key",
    "http.client.idle.timeout.ms": "55000",
    "http.client.response.buffer.size": "2048000",
    "logging.is.verbose": "false",
    "logging.is.signature.included": "false",
    "logging.is.url.body.truncated": "true",
    "sml.request.retry.delay.ms": 10000,
    "sml.request.max.retries": 3,
    "sml.project.must.preexist": "false",
    "sml.create.project.type": "cpu",
    "sml.create.project.params": "",
    "sml.create.project.persist": "false",
    "drop.invalid.message": "false",
    "key.ignore": "true",
    "schema.ignore": "true",
    "behavior.on.malformed.documents": "warn",
    "value.converter.schema.registry.url": "http://host.docker.internal:8081"
  }
}
```
