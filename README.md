
# Kafka #

A Container who has a zookeeper and kafka with connectors installed.
To read data from a oracle database using query flashback tables and publish on a topic KAFKA.


### The container have ###

-- a container for Zookeeper
flinox/zookeeper:v2

-- a container for Kafka / Rest API / KSQL / Connectors
flinox/kafka:v2

### The docker-compose.yml file ###

```
version: "3"
      
networks:
  network-kafka:
    driver: bridge
    
services:

    flinox_zookeeper:
      
      image: flinox/zookeeper:v3
      
      container_name: flinox_zookeeper
      
      hostname: flinox_zookeeper
      
      user: zookeeper
      
      volumes:
       - C:\Temp:/media/storage
      
      ports:
       - "2181:2181"
       - "2888:2888"
       - "3888:3888"

      expose:
       - "2181"
       - "2888"
       - "3888"
       
      environment:
       - AUTO_START=0
       - ZOO_LOG_DIR=/media/storage/zookeeper/log
       - ZOO_LOG4J_PROP=```INFO,ROLLINGFILE```
       - ZOOCFGDIR=/media/storage/zookeeper/config
       
      networks:
       - network-kafka
      
      command: ["/zookeeper_start.sh"]
      

    flinox_kafka:
      
      image: flinox/kafka:v3

      container_name: flinox_kafka      
      
      hostname: flinox_kafka
      
      volumes:
       - C:\Temp:/media/storage
      
      ports:
       - "8081:8081"
       - "8082:8082"
       - "8083:8083"
       - "8088:8088"       
       - "9092:9092"

      expose:
       - "8081"
       - "8082"
       - "8083"
       - "8088"
       - "9092"
       
      environment:
       - AUTO_START=0
       - KAFKA_LOG4J_OPTS=-Dlog4j.configuration=/media/storage/kafka/config/log4j.properties
       - LOG4J_DIR=/media/storage/kafka/config/tools-log4j.properties
       - LOG_DIR=/media/storage/kafka/log
       
      networks:
       - network-kafka       

      depends_on:
       - flinox_zookeeper   
       
      command: ["/kafka_start.sh"]   
```


### The parameters ### 

- AUTO_START to automatically starts the zookeeper / kafka, default is 0;
- Volume, C:\Temp:/media/storage, where C:\Temp must be your local folder and /media/storage is the folder on container, to store data and logs files;


### The folder and the configuration files is on this package, change this files if needed
```
/media_storage/kafka/config/connect-console-sink.properties
/media_storage/kafka/config/connect-console-source.properties
/media_storage/kafka/config/connect-distributed.properties
/media_storage/kafka/config/connect-file-sink.properties
/media_storage/kafka/config/connect-file-source.properties
/media_storage/kafka/config/connect-log4j.properties
/media_storage/kafka/config/connect-standalone.properties
/media_storage/kafka/config/consumer.properties
/media_storage/kafka/config/kafka-rest.properties
/media_storage/kafka/config/ksql-server.properties
/media_storage/kafka/config/log4j.properties
/media_storage/kafka/config/producer.properties
/media_storage/kafka/config/server.properties
/media_storage/kafka/config/tools-log4j.properties
/media_storage/kafka/config/zookeeper.properties
/media_storage/zookeeper/config/configuration.xsl
/media_storage/zookeeper/config/log4j.properties
/media_storage/zookeeper/config/zoo.cfg
/media_storage/zookeeper/data/zookeeper_server.pid
/media_storage/zookeeper/log/zookeeper.log
```

#### The main file to change zookeeper configuration is:
```\media_storage\zookeeper\config\zoo.cfg```

#### The main file to change kafka configuration is:
```
\media_storage\kafka\config\server.properties
\media_storage\kafka\config\producer.properties
\media_storage\kafka\config\kafka-rest.properties
\media_storage\kafka\config\ksql-server.properties
```

### For manual executions without docker-compose.yml ###

### CREATE A NETWORK DOCKER ###
```docker network create -d "bridge" --attachable network_kafka```

### ZOOKEEPER zookeeper-3.4.13 standalone ###
```docker run -it --rm --hostname flinox_zookeeper --name flinox_zookeeper --network network_kafka --env AUTO_START=1 -v "C:\Temp:/media/storage" --env ZOO_LOG_DIR="/media/storage/zookeeper/log" --env ZOO_LOG4J_PROP='INFO,ROLLINGFILE' --env ZOOCFGDIR="/media/storage/zookeeper/config" -u zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 flinox/zookeeper:v2```

### KAFKA 2.11-1.1.0 e KSQL confluente 4.1.1 ( Aguardar subir todos os serviços, Kafka, rest API e KSQL server ) ###
```docker run -it --rm --hostname flinox_kafka --name flinox_kafka --network network_kafka -p 8081:8081 -p 8082:8082 -p 8083:8083 -p 9092:9092 -p 8088:8088 --env AUTO_START=1 --env KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:/media/storage/kafka/config/log4j.properties" --env LOG4J_DIR="/media/storage/kafka/config/tools-log4j.properties" --env LOG_DIR="/media/storage/kafka/log" -v "C:\Temp:/media/storage" flinox/kafka:v2```

### KAFKA nova session para Criar Topico e Produzir mensagens ###
```docker exec -it flinox_kafka bash```

### Create a Topic ###
```./kafka-topics.sh --zookeeper flinox_zookeeper:2181 --create --topic aluno-olimpo --config retention.ms=86400000 --partitions 3 --replication-factor 1```

### Produce Messages, no producer os --property 'parse.key=true' --property 'key.separator=:' que vão determinar a chave e o separador da chave e valor ###
```./kafka-console-producer.sh --broker-list flinox_kafka:9092 --topic aluno-olimpo --property 'parse.key=true' --property 'key.separator=:'```

Exemplo de mensagens JSON:
```
1:{ "id":1 , "nome" : "Fernando Lino D T Silva", "cpf" : "33333333333"}
1:{ "id":1 , "nome" : "Fernando Lino D T Silva", "cpf" : "22222222222"}
1:{ "id":1 , "nome" : "Fernando LDT Silva", "cpf" : "11111111111"}
1:{ "id":1 , "nome" : "Fernando LT Silva", "cpf" : "00000000000"}
2:{ "id":10 , "nome" : "Leonardo Lino", "cpf" : "44444444444"}
2:{ "id":10, "nome" : "Leonardo L DT Silva", "cpf" : "55555555555"}
2:{ "id":10, "nome" : "Leo Lino D T Silva", "cpf" : "11111111111"}
2:{ "id":10, "nome" : "Leonardo Silva", "cpf" : "00000000000"}
```

### Abrir nova session para o KSQL ###
```docker exec -it flinox_kafka bash```

### Acessar pasta do KSQL ###
```cd /opt/confluent-4.1.1/bin/```

### Acessar o KSQL Client ###
```./ksql http://flinox_kafka:8088```

### Testar a conectividade ###
```show properties;```
 
### Create Stream ###
```CREATE STREAM stream_aluno_olimpo (id int, nome varchar, cpf varchar) WITH (kafka_topic='aluno-olimpo', value_format='JSON', KEY=’id’);```

### Describe Stream ###
```DESCRIBE strem_aluno_olimpo;```
 
### Select no Stream, os resultados só são apresentados no momento que são enviados.
```select * from stream_aluno_olimpo;```

###### Some problems to run KSQL because of version 10 of JAVA ######

### Ajusts needed ###
```
cd /opt/confluent-4.1.1/bin

vi ksql-run-class.sh
```

### Remove the parameters below ###
```
-XX:+UseParNewGC
-PrintGCDateStamps
-XX:+PrintGCDetails
-UseGCLogFileRotation
```
