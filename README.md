## Installation Steps:
## Kafka Cluster Setup:
1) Install Docker

2)  Launch Kafka cluster based  on  landoop image in docker-compose file since landoop image has already loaded lot of connectors in the image. Run this command after downloading docker-compose file from here:

docker-compose.yml

 

version: '2'<br>
services:<br>
  //this is our kafka cluster.<br>
  kafka-cluster:<br>
    image: landoop/fast-data-dev:cp3.3.0<br>
    environment:<br>
      ADV_HOST: 127.0.0.1        # Change to 192.168.99.100 if using Docker Toolbox<br>
      RUNTESTS: 0                 # Disable Running tests so the cluster starts faster<br>
    ports:
      - 2181:2181                 # Zookeeper<br>
      - 3030:3030                 # Landoop UI<br>
      - 8081-8083:8081-8083       # REST Proxy, Schema Registry, Kafka Connect ports<br>
      - 9581-9585:9581-9585       # JMX Ports<br>
      - 9092:9092                 # Kafka Broker<br>

 

Start kafka Cluster  by the command:

docker-compose up kafka-cluster

 

## Kafka Rest Proxy:

Kafka Rest Proxy is an open source project created by Confluent. It highly useful for the clients which doesn't support Kafka libraries

It is integrated with Schema Registry in case producer and consumer want to use avro:

Producer can publish data to Kafka Broker in 3 ways:

1) Binary: To send the data in binary form, it should be base 64 encoded first.

2)  Json: Json data can be send as it is, any valid json works only issue here is no validation with respect to use case.

3) Avro: REST proxy has native support for avro and it is directly connected to Schema Registry. In avro, producer need to send schema(stringified json) along with json. If there is json which is not satisfying schema present in schema registry then that request will fail immediately 

## Producing with the Json
URI: http://{{ hostname  }}:8082/topics/{topic-name}
Header: Content-Type: application/vnd.kafka.binary.v2+json .
Body:
{
"records": [{
"value": {
"source": "abc",
"customerId": "xyz",
}
}

]

}

 

 

Response:

{
"offsets": [
{
"partition": 2,
"offset": 0,
"error_code": null,
"error": null
},
{
"partition": 1,
"offset": 0,
"error_code": null,
"error": null
}
],
"key_schema_id": null,
"value_schema_id": null
}


 

## Producing with the Avro:

URI: http://{{ hostname  }}:8082/topics/{topic-name}
Header: Content-Type: application/vnd.kafka.avro.v2+json
Body:
{
"value_schema": "{\"type\": \"record\", \"name\": \"User\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}, {\"name\" :\"age\", \"type\": [\"null\",\"int\"]}]}",
"records": [
{
"value": {"name": "Virat", "age": {"int": 30} }
},
{
"value": {"name": "Rahul", "age": {"int": 41} }
}
]
}

 

 

Response:

{
"offsets": [
{
"partition": 0,
"offset": 6,
"error_code": null,
"error": null
},
{
"partition": 0,
"offset": 7,
"error_code": null,
"error": null
}
],
"key_schema_id": null,
"value_schema_id": 330
}

After 1st request , pass value_schema_id as present in response of 1st request instead of entire json schema, for example
{
"value_schema_id": 330,
"records": [
{
"value": {"name": "testUser6", "age": null }
},
{
"value": {"name": "testUser7", "age": {"int": 25} },
"partition": 0
}
]
}


## Note:
Using Kafka REST proxy instead of kafka native protocol decreases the throughput 3-4 times.<br>
Use of Avro is recommended, if language support it as it perform the schema validation during POST request to REST proxy and failed immediately in case of schema mismatch
 
