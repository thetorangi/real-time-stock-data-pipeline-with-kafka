# Real Time Stock Data Pipeline With Kafka 

- working on it, Will Upload Soon, Be connected :) <3

try this 
wget https://downloads.apache.org/kafka/3.3.1/kafka_2.12-3.3.1.tgz

else 
wget https://archive.apache.org/dist/kafka/3.3.1/kafka_2.12-3.3.1.tgz


tar -xvf kafka_2.12-3.3.1.tgz

-----------------------
java -version
sudo yum install java-1.8.0-openjdk
java -version
cd kafka_2.12-3.3.1

Start Zoo-keeper:
-------------------------------
bin/zookeeper-server-start.sh config/zookeeper.properties

Open another window to start kafka
But first ssh to to your ec2 machine as done above


Security Group and Make changes 


bin/kafka-topics.sh --create --topic demo_test --bootstrap-server

bin/kafka-topics.sh --create --topic demo_test --bootstrap-server EC2-ip:9092 --replication-factor 1 --partitions 1

bin/kafka-console-producer.sh --topic demo_test --bootstrap-server Ec2-ip:9092

bin/kafka-console-consumer.sh --topic demo_test --bootstrap-server Ec2-ip:9092






ssh -i "kafka-stock.pem" ec2-user@ec2-13-233-85-49.ap-south-1.compute.amazonaws.com
