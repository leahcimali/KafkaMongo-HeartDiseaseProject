# KafkaMongo-HeartDiseaseProject
Creation of a message Gateaway with Kafka for an hospital

## DOCKER 

# Creating the cluster : 
sudo docker-compose up --build -d 

# Connecting to containers :

Kafka : 

sudo docker exec -it kafka bash

Mongo : 

sudo docker exec -it mongo bash

Connect : 

sudo docker exec -it connect bash

# To use the control-center API :

http://localhost:9021/

# Creation and configuration of Topic and connector can be done with concrol-center API or Manualy

## TOPIC CREATION

## WITH CONTROL-CENTER API

Go to Cluster on the left bar -> Topics -> Add a Topic

Here we are creating 3 topics : 

"MaladeUrgence", "NonMalade", "NonMaladeSurveillance" each with number of partition = 2.

## MANUALLY 

connect to kafka container : sudo docker exec -it kafka bash

/usr/bin/kafka-topics --create --topic MaladeUrgence --bootstrap-server localhost:9092 --partitions 2 --replication-factor 1

/usr/bin/kafka-topics --create --topic NonMaladeSurveillance --bootstrap-server localhost:9092 --partitions 2 --replication-factor 1

/usr/bin/kafka-topics --create --topic NonMalade --bootstrap-server localhost:9092 --partitions 2 --replication-factor 1

## MONGO DATABASE

connect to mongodb container

sudo docker exec -it mongo bash

connect to mongo as root user

mongo --host localhost -u root -p root

Create Database Hopital and a collection MaladeUrgence : 

use Hospital

db.createCollection(‘MaladeUrgence’)

The collection is empty, see with : 

db.MaladeUrgence.find()

## MONGO SINK CONNECTOR CONFIGURATION

## WITH CONTROL-CENTER API

Go to Cluster on the left bar -> Connect -> click on cluster-name connect-default -> Add Connector -> choose mongosinkconnector

Choose topic name : MaladeUrgence 

Choose Connector name : UrgenceConnector

Task max : 1

Key Convertor Class: org.apache.kafka.connect.storage.StringConverter

Value Convertor Class : org.apache.kafka.connect.storage.StringConverter

MongoDb Connection URI : mongodb://root:root@mongo:27017

The MongoDb Database Name: Hopital

The default MongoDB collection name : MaladeUrgence

Continue -> Launch -> The connector is created ! 


## MANUALLY WITH REST API:

sudo docker exec -it connect bash

curl -X POST -H "Content-Type: application/json" --data '
  {"name": "UrgenceConnector",
   "config": {
     "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
     "tasks.max":"1",
     "topics":"MaladeUrgence",
     "connection.uri":"mongodb://root:root@mongo:27017",
     "database":"Hopital",
     "collection":"MaladeUrgence",
     "key.converter":"org.apache.kafka.connect.storage.StringConverter",
     "key.converter.schemas.enable":false,
     "value.converter":"org.apache.kafka.connect.storage.StringConverter",
     "value.converter.schemas.enable":false
 }}' http://localhost:8083/connectors -w "\n"

## CONNECTING TO TOPICS : 

On 3 separate shell connect to kafka then : 

Shell 1 : /usr/bin/kafka-console-producer --topic MaladeUrgence --broker-list localhost:9092

Shell 2: /usr/bin/kafka-console-producer --topic NonMaladeSurveillance --broker-list localhost:9092

Shell 3 : /usr/bin/kafka-console-producer --topic NonMalade --broker-list localhost:9092

Don't shut those, we have to wait for the producer to produce message! 

## CONNECTORS : 
Two connectors where create with python-kafka :

Consumer Surveillance.ipynb

Consumer Malade.ipynb


## FOR PRODUCER : 

The producer was create with python-Kafka on jupyter notebook. 

You can manually write message on kafka with:

/usr/bin/kafka-console-producer --topic <topic-name> --broker-list localhost:9092

<topic-name> is the name of the topic without brackets

The python producer : 
Producer.ipynb

Wait at least 1 or 2 minutes. 

Veryfiying the database : 

connect to mongo as root user

mongo --host localhost -u root -p root

use Hospital

db.MaladeUrgence.find()

Entries have been created in the database ! 
GG ! Great job! 

