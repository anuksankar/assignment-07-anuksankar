## W205 - Assignment 7 - Setup Pipeline

### Spark Stack with Kafka

Setup - As we have already created a directory ~/w205/spark-with-kafka in class, just cd into it for the assignment.
```
cd ~/w205/spark-with-kafka

cp ~/w205/course-content/07-Sourcing-Data/docker-compose.yml .
```

Spin up the cluster in one session and then check the logs in another session.
```
docker-compose up -d

docker-compose logs -f kafka
```

Create a topic - "assessment-attempts"
```
docker-compose exec kafka kafka-topics --create --topic assessment-attempts --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

The following is displayed:
```
	Created topic "assessment-attempts".
```
Check the topic
```
docker-compose exec kafka kafka-topics --describe --topic assessment-attempts --zookeeper zookeeper:32181
```
The following is displayed:
```
	Topic:assessment-attempts       PartitionCount:1        ReplicationFactor:1    Configs:
        Topic: assessment-attempts      Partition: 0    Leader: 1       Replicas: 1     Isr: 1
```
Check out messages
```
docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json"

docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.'"

docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c"
```
Publish messages
```
docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessment-attempts && echo 'Produced 100 messages.'"
```
The following is displayed
```
	Produced 100 messages.
```

Run spark using the spark container
```
docker-compose exec spark pyspark
```
starts Spark version 2.2.0

Read from kafka, at the pyspark prompt
```
messages = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","assessment-attempts").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
```

See the schema
```
messages.printSchema()
```
The following is displayed
```
root
 |-- key: binary (nullable = true)
 |-- value: binary (nullable = true)
 |-- topic: string (nullable = true)
 |-- partition: integer (nullable = true)
 |-- offset: long (nullable = true)
 |-- timestamp: timestamp (nullable = true)
 |-- timestampType: integer (nullable = true)
```

See the messages
```
messages.show()
```
The following is displayed
```
+----+--------------------+-------------------+---------+------+--------------------+-------------+
| key|               value|              topic|partition|offset|           timestamp|timestampType|
+----+--------------------+-------------------+---------+------+--------------------+-------------+
|null|[7B 22 6B 65 65 6...|assessment-attempts|        0|     0|1969-12-31 23:59:...|            0|
|null|[7B 22 6B 65 65 6...|assessment-attempts|        0|     1|1969-12-31 23:59:...|            0|
|null|[7B 22 6B 65 65 6...|assessment-attempts|        0|     2|1969-12-31 23:59:...|            0|
...................................................................................................
...................................................................................................
+----+--------------------+-------------------+---------+------+--------------------+-------------+
only showing top 20 rows
```
Cast as strings
```
messages_as_strings=messages.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
```
Take a look at the values
```
messages_as_strings.show()
```
Results
+----+--------------------+
| key|               value|
+----+--------------------+
|null|{"keen_timestamp"...|
|null|{"keen_timestamp"...|
...........................
...........................
|null|{"keen_timestamp"...|
+----+--------------------+
only showing top 20 rows
```

```
messages_as_strings.printSchema()
```
Results
```
root
 |-- key: string (nullable = true)
 |-- value: string (nullable = true)
```

```
messages_as_strings.count()
```
Result
```
3280
```

Unrolling json

```
messages_as_strings.select('value').take(1)

messages_as_strings.select('value').take(1)[0].value

import json

first_message=json.loads(messages_as_strings.select('value').take(1)[0].value)

first_message

print(first_message['sequences']['counts']['total'])
4

print(first_message['sequences']['questions'][0]['options'][0]['checked'])
True

print(first_message['sequences']['questions'][0]['options'][0]['at'])
2018-01-23T14:23:24.670Z
```


