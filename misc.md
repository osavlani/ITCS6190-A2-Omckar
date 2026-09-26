docker --version
java -version
mvn -version

docker compose -f docker-compose.codespaces.yml up -d

docker ps

mvn clean package

docker cp target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/tmp/
docker cp shared-folder/input/data/small_dataset.txt resourcemanager:/tmp/
docker cp shared-folder/input/data/dataset.txt resourcemanager:/tmp/

docker exec -it resourcemanager bash
cd /tmp

hadoop fs -mkdir -p /input/data
hadoop fs -put ./small_dataset.txt /input/data
hadoop fs -put ./dataset.txt /input/data
hadoop fs -ls /input/data

docker network ls
name nodes were failing, retried with H4 cosedspaces.yml file

hadoop jar /tmp/DocumentSimilarity-0.0.1-SNAPSHOT.jar \
  com.example.controller.DocumentSimilarityDriver /input/data/small_dataset.txt /output/small_dataset

hadoop fs -cat /output/small_dataset/* :
bash-4.2$ hadoop fs -cat /output/small_dataset/*
Document1, Document2 Similarity: 0.18
Document1, Document3 Similarity: 0.20
Document2, Document3 Similarity: 0.10

hadoop fs -cat /output/dataset/*

hdfs dfs -get /output /tmp/
exit

docker cp resourcemanager:/tmp/output/. shared-folder/output/

docker compose -f docker-compose.codespaces.yml down


Use of gen AI to help design the 3 java functions.