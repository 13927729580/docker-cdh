# loicmathieu/cloudera-cdh
Base image with Cloudera CDH installed on it.
It's purpose is to be the base to make more specific CDH container, so don't use it directly but instead use

loicmathieu/cloudera-cdh-namenode : will launch an HDFS namenode and an HDFS secondarynamenode
loicmathieu/cloudera-cdh-datanode : will launch and HDFS datanode
But if you really want to use it, OK, you can :

docker pull loicmathieu/cloudera-cdh
docker run -ti loicmathieu/cloudera-cdh
This should print out the Hadoop version (hadoop version command will be launched)

# loicmathieu/cloudera-cdh-namenode
A container running Cloudera CDH HDFS Namenode and SecondaryNamenode

The tag latest is CDH 5.15 but CDH 6 is available via the tag cdh-6.0.0

Disclaimer : If you want a container shipped with all the Hadoop components in it, take a look at the cloudera/quickstart container. If you want to be able to run multiple container, each with a single hadoop role, use the loicmathieu/cloudera-cdh-<role> containers provided here

This container is derived from loicmathieu/cloudera-cdh and will setup an HDFS Namenode and a HDFS SecondaryNamenode. To use it, you will need a Datanode that can be run with loicmathieu/cloudera-hdfs-datanode.

A full example of how to use it with the other Hadoop component can be found in the edgenode documentation : https://hub.docker.com/r/loicmathieu/cloudera-cdh-edgenode/

The Namenode will expose it's 8020 port, to use it, you will then need to start a Datanode and make sure the network stack is OK so that the Namenode and the Datanode can communicate together.

The container will use supervisor to start both the Namenode and the SecondaryNamenode.

The logs are located in the /var/log/hadoop-hdfs directory for both the Namenode and the SecondaryNamenode.

Running the container

docker pull loicmathieu/cloudera-cdh-namenode
docker run -d -p 8020:8020 loicmathieu/cloudera-cdh-namenode
Running the cluster

create a network :
docker network create hadoop
start the yarn master :
docker run -d --net hadoop --net-alias namenode \
-p 8020:8020 loicmathieu/cloudera-cdh-namenode
start the datanode :
docker run -d --net hadoop --net-alias datanode1 --link namenode \
-p 50020:50020 loicmathieu/cloudera-cdh-datanode
To test the installation, connect to the namenode container, check the logs and if everything is fine make some HDFS operations, for example :

docker exec -ti <namenode_id> bash
hadoop fs -put /var/log/hadoop-hdfs/* /
hadoop fs -ls /
For a more complex cluster setup including HDFS, Yarn/MaprReduce, Hive, Pig, Spark, ... see loicmathieu/cloudera-cdh-edgenode that put all this together

# loicmathieu/cloudera-cdh-datanode
A container running Cloudera Hadoop CDH HDFS Datanode and Yarn NodeManager

The tag latest is CDH 5.15 but CDH 6 is available via the tag cdh-6.0.0

Disclaimer : If you want a container shipped with all the Hadoop components in it, take a look at the cloudera/quickstart container. If you want to be able to run multiple container, each with a single hadoop role, use the loicmathieu.cloudera-cdh-<role> containers provided here

This container is derived from loicmathieu/cloudera-cdh and will setup an HDFS datanode and a Yarn NodeManager.

A full example of how to use it with the other Hadoop component can be found in the edgenode documentation : https://hub.docker.com/r/loicmathieu/cloudera-cdh-edgenode/

The datanode will expose it's 50020 and 50075 ports, to use it, you first need to start a namenode
(using loicmathieu/cloudera-hdfs-namenode) and make sure the network stack is OK so that the namenode and datanode can communicate together.

The nodemanager will expose it's 8042 port. If you want to use it, you also need to start a loicmathieu/cloudera-cdh-yarnmaster container and you need to configure the hostname of your container (using the docker run -h option) because communication between the nodemanger and the ressourcemanager are based on the hostname.

The container will use supervisor to start both the Datanode and the NodeManager.

The Datanode logs are in /var/log/hadoop-hdfs and the NodeManager logs are in /var/log/hadoop-yarn

Running the container

docker pull loicmathieu/cloudera-cdh-datanode
docker run -d -p 50020:50020 -p 50075:50075 -p 8042:8042 -h datanode1 loicmathieu/cloudera-cdh-datanode
Running the cluster

create a network :
docker network create hadoop
start the namenode :
docker run -d --net hadoop --net-alias namenode \
-p 8020:8020 loicmathieu/cloudera-cdh-namenode
start the yarn manager :
docker run -d --net hadoop --net-alias yarnmaster \
-p 8032:8032 -p 8088:8088 loicmathieu/cloudera-cdh-yarnmaster
start the datanode :
docker run -d --net hadoop --net-alias datanode1 -h datanode1 --link namenode --link yarnmaster \
-p 50020:50020 -p 50075:50075 -p 8042:8042 loicmathieu/cloudera-cdh-datanode
To test the installation, connect to the namenode or the yarnmaster, check the logs and if everything is fine make some HDFS operation, for example :

docker exec -ti <namenode_id> bash
hadoop fs -ls /
hadoop fs -put /var/log/hadoop-hdfs/* /
hadoop fs -ls /
or some Yarn/MapReduce operation

hadoop fs -put /var/log/hadoop-hdfs/* /
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar \
wordcount /hadoop-hdfs* /wordcount
hadoop fs -ls /wordcount
For a more complex cluster setup including HDFS, Yarn/MaprReduce, Hive, Pig, Spark, ... see loicmathieu/cloudera-cdh-edgenode that put all this together

# loicmathieu/cloudera-cdh-edgenode
A container installed with Cloudera Hadoop CDH and aims to be a client for HDFS and Yarn/Mapreduce.

The tag latest is CDH 5.15 but CDH 6 is available via the tag cdh-6.0.0

This container is linked to the other loicmathieu/cloudera-cdh-* containers and aims to be the edge node so the entry point to send files, launch scripts and jobs, etc. to the cluster.

This container contains the following Hadoop client : hdfs, yarn, mapreduce v2, pig, hive, spark, sqoop, flume.

Some example of how to run it :

If not done already, pull all the needed images

docker pull loicmathieu/cloudera-cdh-namenode
docker pull loicmathieu/cloudera-cdh-yarnmaster
docker pull loicmathieu/cloudera-cdh-datanode
docker pull loicmathieu/cloudera-cdh-edgenode
First, setup a network for the cluster

docker network create hadoop
Then start the HDFS and Yarn master containers

docker run -d --net hadoop --net-alias namenode \
-p 8020:8020 loicmathieu/cloudera-cdh-namenode
docker run -d --net hadoop --net-alias yarnmaster \
-p 8032:8032 -p 8088:8088 loicmathieu/cloudera-cdh-yarnmaster
Then start a datanode, warning : the hostname of the datanode needs to be specified in order yarn RessourceManager to be able to communicate with it (use of the -h docker run option).

docker run -d --net hadoop --net-alias datanode1 -h datanode1 \
--link namenode --link yarnmaster -p 50020:50020 -p 50075:50075 -p 8042:8042 \
loicmathieu/cloudera-cdh-datanode
Finally launch the edge node and connect to it

docker run -ti --net hadoop --net-alias edgenode --link namenode --link yarnmaster \
loicmathieu/cloudera-cdh-edgenode bash
Optionnaly, you can mount the /staging volume to be able to easily send/get data to/from the cluster. It can facilitate putting stuff on HDFS or sending JAR files to Yarn.

Some example of how to run it :
The container include test data and scripts to test the cluster, here is a small snippet of what can be done :

HDFS & MapReduce :

 hadoop fs -mkdir /cities
 hadoop fs -put cities.csv /cities
 hadoop fs -cat /cities/cities.csv
 hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar \
 wordcount /cities/cities.csv /wordcount
 hadoop fs -ls /wordcount
Pig :

 pig cities.pig
 hadoop fs -ls /data_by_department
 hadoop fs -cat /data_by_department/part-r-00000
Hive :

 beeline -u jdbc:hive2:// -f cities.hql
 beeline -u jdbc:hive2://
 select * from cities limit 10;
 select * from cities where department = '82' limit 10;
Spark (local) :

spark-shell
val cities = sc.textFile("hdfs:///cities");
cities.count();
exit;
Spark (yarn) :

spark-shell --master yarn
val cities = sc.textFile("hdfs:///cities");
cities.count();
exit;
Flume :
There's an embeded flume tests that can be used with the loicmathieu/apache-httpd-flume container (https://hub.docker.com/r/loicmathieu/apache-httpd-flume). This container contains and apache httpd webserver and a flume agent that will tail the apache access log and send it to a flume collector edgenode:9000. You can see the configuration file /flume_httpd.conf

On the edgenode, the flume conf is also in /flume_httpd.conf and there is a /start_flume.sh script that will launch an agent that will listen on port 9000 and write to hdfs://users/root/collector the data written to it.

Here are the step to test it :

docker pull loicmathieu/apache-httpd-flume
docker run -d --net hadoop --net-alias flume -p 80:80 loicmathieu/apache-httpd-flume
docker run -ti --net hadoop --net-alias edgenode --link namenode --link yarnmaster loicmathieu/cloudera-cdh-edgenode bash
start_flume.sh
Here, I forward the port 80 of the loicmathieu/apache-httpd-flume container to the port 80 of the host, accessing it via a browser will write lines to the apache httpd access logs that will be send to the flume agent of the edgenode and written to HDFS

hadoop fs -ls /user/root/collector
Sqoop :
It's installed, use it with scoop from the shell ...

# loicmathieu/cloudera-cdh-yarnmaster
A container running Cloudera CDH HDFS Namenode and SecondaryNamenode

The tag latest is CDH 5.15 but CDH 6 is available via the tag cdh-6.0.0

Disclaimer : If you want a container shipped with all the Hadoop components in it, take a look at the cloudera/quickstart container. If you want to be able to run multiple container, each with a single hadoop role, use the loicmathieu.cloudera-cdh-<role> containers provided here

This container is derived from loicmathieu/cloudera-cdh and will setup a Yarn ResourceManager and a HDFS MaprReduce 2 HistoryServer. To use it, you will need a Datanode that can be run with loicmathieu/cloudera-hdfs-datanode.

A full example of how to use it with the other Hadoop component can be found in the edgenode documentation : https://hub.docker.com/r/loicmathieu/cloudera-cdh-edgenode/

The ResourceManager will expose it's 8032 port, to use it, you will then need to start a Datanode and make sure the network stack is OK so that the ResourceManager and the Datanode can communicate together.

The History server will expose it's 8080 port.

The container will use supervisor to start both the ResourceManager and the HistoryServer.

The logs are located in the /var/log/hadoop-yarn directory for the ResourceManager and the /var/log/hadoop-mapreduce directoy for the HistoryServer

Running the container

docker pull loicmathieu/cloudera-cdh-yarnmaster
docker run -d -p 8032:8032 -p 8088:8088 loicmathieu/cloudera-cdh-yarnmaster
Running the cluster

create a network :
docker network create hadoop
start the yarn master :
docker run -d --net hadoop --net-alias yarnmaster \
-p 8032:8032 -p 8088:8088 loicmathieu/cloudera-cdh-yarnmaster
start the datanode :
docker run -d --net hadoop --net-alias datanode1 -h datanode1 --link yarnmaster \
-p 50020:50020 -p 50075:50075 -p 8042:8042 loicmathieu/cloudera-cdh-datanode
To test the installation, connect to the ResourceManager, check the logs and if everything is fine make some Yarn operation, for example :

docker exec -ti <yarnmaster_id> bash
hadoop fs -put /var/log/hadoop-yarn/* /
hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount \
hadoop-yarn* /wordcount
hadoop fs -ls /wordcount
For a more complex cluster setup including HDFS, Yarn/MaprReduce, Hive, Pig, Spark, ... see loicmathieu/cloudera-cdh-edgenode that put all this together

# https://hub.docker.com/u/loicmathieu/
