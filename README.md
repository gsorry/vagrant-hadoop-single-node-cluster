# Vagrant Hadoop Single-node Cluster

Start Vagrant VM:

```
vagrant up
vagrant ssh
```

Install Java:

```
sudo apt-get install openjdk-8-jdk -y
export JAVA_HOME=`dirname $(dirname $(readlink -f $(which javac)))`
```

Setup SSH:

```
sudo apt-get install ssh -y
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

Install Hadoop and Spark:

```
curl -O https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
curl -O https://downloads.apache.org/spark/spark-2.4.5/spark-2.4.5-bin-without-hadoop.tgz

tar -xvzf hadoop-3.2.1.tar.gz
tar -xvzf spark-2.4.5-bin-without-hadoop.tgz

mv hadoop-3.2.1 hadoop
mv spark-2.4.5-bin-without-hadoop spark

rm hadoop-3.2.1.tar.gz
rm spark-2.4.5-bin-without-hadoop.tgz

export HADOOP_HOME=~/hadoop
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export PATH=${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```

Setup Hadoop:

```
nano ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
```

```
export JAVA_HOME=`dirname $(dirname $(readlink -f $(which javac)))`
```

```
nano ${HADOOP_HOME}/etc/hadoop/core-site.xml
```

```
# <configuration>
#     <property>
#         <name>fs.defaultFS</name>
#         <value>hdfs://localhost:9000</value>
#     </property>
# </configuration>
```

```
nano ${HADOOP_HOME}/etc/hadoop/hdfs-site.xml
```

```
# <configuration>
#     <property>
#         <name>dfs.replication</name>
#         <value>1</value>
#     </property>
# </configuration>
```

```
hdfs namenode -format
${HADOOP_HOME}/sbin/start-dfs.sh
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/vagrant
```

Brose:

[http://192.0.2.20:9870/](http://192.0.2.20:9870/)

```
${HADOOP_HOME}/sbin/stop-dfs.sh
```

```
nano ${HADOOP_HOME}/etc/hadoop/mapred-site.xml
```

```
# <configuration>
#     <property>
#         <name>mapreduce.framework.name</name>
#         <value>yarn</value>
#     </property>
#     <property>
#         <name>mapreduce.application.classpath</name>
#         <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
#     </property>
# </configuration>
```

```
nano ${HADOOP_HOME}/etc/hadoop/yarn-site.xml
```

```
# <configuration>
#     <property>
#         <name>yarn.nodemanager.aux-services</name>
#         <value>mapreduce_shuffle</value>
#     </property>
#     <property>
#         <name>yarn.nodemanager.env-whitelist</name>
#         <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
#     </property>
# </configuration>
```

```
${HADOOP_HOME}/sbin/start-yarn.sh
```

Browse:

[http://192.0.2.20:8088/](http://192.0.2.20:8088/)

```
${HADOOP_HOME}/sbin/stop-yarn.sh
```
