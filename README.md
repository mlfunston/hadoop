## Hadoop 2.7.3
### With Debian Jessie, CentOS 7, Centos 6.8
### and Alpine Linux (466.6 MB)
#### · Pseudo distributed mode
#### · Fully distributed mode
#### · PySpark with Jupyter Notebook
#### . RStudio Server
#### . Ecosystem:
##### . Hadoop, Spark, Zookeeper, HBase, Hive, Pig, Sqoop, Flume, Mahout
##### . JDBC implemented and ready for sqoop and spark
##### . Instances of MariaDB and Oracle 11g
-----
### Script for your cluster from 1 to 9 nodes.
```
curl -L https://raw.githubusercontent.com/luvres/hadoop/master/zoneCluster.sh -o ~/zoneCluster.sh
alias zoneCluster="bash ~/zoneCluster.sh"
```
#### Create a directory for notebooks and Include directory created above on flag "-v"
```
mkdir $HOME/notebooks
```
### Create cluster of a node 
#### The total of 2, as the namenode assumes one more node
```
zoneCluster
```
### Hadoop Browser
```
http://localhost:8088

http://localhost:50070
```
#### Access by Jupyter Notebook
```
http://localhost:8888/terminals/1

 sh-4.2# bash <enter>
```
### To create a cluster of maximum 9 nodes (10 including the namenode)
```
zoneCluster 3

docker logs -f Hadoop
```
##### Note: The script is limited to a maximum of 9 nodes because multiple hosts are being created on only one host and I see no point in overloading your machine. The settings are ready for a real cluster and in the future I want to create scripts for provisioning with docker swarm.

##### Options: { stop | start | remove | Stop | pseudo | cos6 | cos7 | alpine }

##### Stop and Remove the cluster
```
zoneCluster Stop
```
### Raise Cluster with Mariadb and Oracle 11g Database
```
zoneCluster 2 -db
```
#### Import data from Mariadb with Sqoop
```
sqoop import \
	--connect jdbc:mysql://mariadb:3306/mysql \
	--username root \
	--password maria \
	--table user --m 1
```
#### Checking imported data for the hdfs
```
hdfs dfs -ls -R user
```
### PySpark with Jupyter Notebook
#### Browser access
```
http://localhost:8888
```
#### Spark management jobs
```
http://localhost:4040
```
### RStudio Server
#### Browser access
```
http://localhost:8787

username: root
password: root
```
### Creates a pseudo-distributed instance
```
zoneCluster pseudo
```
#### Equivalent to the command
```
  docker run --rm --name Hadoop -h hadoop \
  -p 8088:8088 -p 8042:8042 -p 50070:50070 -p 8888:8888 -p 4040:4040 \
  -v $HOME/notebooks:/root/notebooks \
  -ti izone/hadoop:ecosystem bash
```
-----
### Pull image latest (with Debian 8)
```
docker pull izone/hadoop
```
#### Run pulled image (Optional flag "-test" to start with a PI test)
```
docker run --rm --name Hadoop -h hadoop \
	-p 8088:8088 \
	-p 8042:8042 \
	-p 50070:50070 \
	-ti izone/hadoop -test bash
```
#### Pull image with CentOS 7
```
docker pull izone/hadoop:cos7
```
#### Pull image with CentOS 6
```
docker pull izone/hadoop:cos6
```
### Pull reduced image with Alpine (466.6 MB)
```
docker pull izone/hadoop:alpine
```
#### Run pulled image (Optional flag "-test" to start with a PI test)
```
docker run --rm --name Hadoop -h hadoop \
	-p 8088:8088 \
	-p 8042:8042 \
	-p 50070:50070 \
	-ti izone/hadoop:alpine -test bash
```
-----
## Examples:

### Hadoop Map Reduce

#### Create a Directory
```
hdfs dfs -mkdir /bigdata
```
#### List diretory
```
hadoop fs -ls /
```
#### Download a file csv
```
wget -c http://compras.dados.gov.br/contratos/v1/contratos.csv
```
#### Copy file to the HDFS directory created above
```
hadoop fs -copyFromLocal contratos.csv /bigdata
```
#### Read file
```
hadoop fs -cat /bigdata/contratos.csv
```
#### Test word count with mapreduce
```
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /bigdata/contratos.csv /output
```
#### Read result
```
hdfs dfs -cat /output/*
```

### Spark MapReduce

#### pyspark jupyter notebook
```
http://localhost:8888/
new -> python
```
#### Terminal commands executed with "!" Straight into the notebook
#### It is the same as running direct on the terminal
```
!mkdir datasets
!curl -L http://www.gutenberg.org/files/11/11-0.txt -o datasets/book.txt
!hdfs dfs -mkdir -p /spark/input
!hdfs dfs -put datasets/book.txt /spark/input
!hdfs dfs -ls /spark/input
```
#### Examples of http://spark.apache.org/examples.html
```
text_file = sc.textFile("hdfs://localhost:9000/spark/input/book.txt")

counts = text_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b)

counts.saveAsTextFile("hdfs://localhost:9000/spark/output"
```
#### View result
```
!hdfs dfs -ls /spark/output
!hdfs dfs -cat /spark/output/part-00000
```
-----
### Pull image with Anaconda
```
docker run --rm --name Hadoop -h hadoop \
	-p 8088:8088 \
	-p 8042:8042 \
	-p 50070:50070 \
	-p 8888:8888 \
	-p 4040:4040 \
	-v $HOME/notebooks:/root/notebooks \
	-ti izone/hadoop:anaconda bash
```
### Pull image with RStudio
```
docker run --rm --name Hadoop -h hadoop \
        -p 8088:8088 \
        -p 8042:8042 \
        -p 50070:50070 \
        -p 8888:8888 \
        -p 4040:4040 \
        -p 8787:8787 \
        -v $HOME/notebooks:/root/notebooks \
        -ti izone/hadoop:rstudio bash
```
-----
### AUTO CONSTRUCTION creation sequence that are in the Docker Hub

### Debian 8
```
git clone https://github.com/luvres/hadoop.git
cd hadoop

docker build -t izone/hadoop . && \
docker build -t izone/hadoop:anaconda ./anaconda/ && \
docker build -t izone/hadoop:rstudio ./rstudio/ && \
docker build -t izone/hadoop:ecosystem ./ecosystem/ && \
docker build -t izone/hadoop:cluster ./cluster/ && \
docker build -t izone/hadoop:datanode ./cluster/datanode
```
### CentOS 7
```
git clone https://github.com/luvres/hadoop.git
cd hadoop

docker build -t izone/hadoop:cos7 ./centos7/ && \
docker build -t izone/hadoop:cos7-miniconda ./centos7/miniconda/ && \
docker build -t izone/hadoop:cos7-ecosystem ./centos7/ecosystem/ && \
docker build -t izone/hadoop:cos7-anaconda ./centos7/anaconda/ && \
docker build -t izone/hadoop:cos7-mahout ./centos7/mahout/ && \
docker build -t izone/hadoop:cos7-cluster ./centos7/cluster/ && \
docker build -t izone/hadoop:cos7-datanode ./centos7/cluster/datanode/

```
### CentOS 6
```
git clone https://github.com/luvres/hadoop.git
cd hadoop

docker build -t izone/hadoop:cos6 ./centos6/ && \
docker build -t izone/hadoop:cos6-miniconda ./centos6/miniconda/ && \
docker build -t izone/hadoop:cos6-ecosystem ./centos6/ecosystem/ && \
docker build -t izone/hadoop:cos6-anaconda ./centos6/anaconda/ && \
docker build -t izone/hadoop:cos6-rstudio ./centos6/rstudio/ && \
docker build -t izone/hadoop:cos6-mahout ./centos6/mahout/ && \
docker build -t izone/hadoop:cos6-cluster ./centos6/cluster/ && \
docker build -t izone/hadoop:cos6-datanode ./centos6/cluster/datanode
```
### Alpine
```
git clone https://github.com/luvres/hadoop.git
cd hadoop

docker build -t izone/hadoop:alpine ./alpine/ && \
docker build -t izone/hadoop:alpine-datanode ./alpine/datanode/
```
