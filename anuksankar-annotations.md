
docker-compose exec kafka kafka-topics --create --topic assessment-attempts --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
	Created topic "assessment-attempts".


docker-compose exec kafka kafka-topics --describe --topic assessment-attempts --zookeeper zookeeper:32181
	Topic:assessment-attempts       PartitionCount:1        ReplicationFactor:1    Configs:
        Topic: assessment-attempts      Partition: 0    Leader: 1       Replicas: 1     Isr: 1



docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json"

docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.'"

docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c"

docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t assessment-attempts && echo 'Produced 100 messages.'"
	Produced 100 messages.


docker-compose exec spark pyspark
	starts Spark version 2.2.0

