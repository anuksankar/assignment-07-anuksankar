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


docker-compose exec spark pyspark
	starts Spark version 2.2.0

