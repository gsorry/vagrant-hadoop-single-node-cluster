# Vagrant Hadoop Single-node Cluster

Start Vagrant VM:

```bash
vagrant up
vagrant ssh
```

Install Java:

```bash
sudo apt-get install openjdk-8-jdk -y
```

Test Java:

```bash
java -version
```

Check Java Home:

```bash
dirname $(dirname $(readlink -f $(which javac)))
```

Setup SSH:

```bash
sudo apt-get install ssh -y
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

Test SSH:

```bash
ssh localhost
```

Set Python3:
```bash
sudo ln -s /usr/bin/python3 /usr/bin/python
```

Install Hadoop and Spark:

```bash
curl -O https://downloads.apache.org/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
curl -O https://downloads.apache.org/spark/spark-2.4.5/spark-2.4.5-bin-without-hadoop.tgz

tar -xvzf hadoop-3.2.1.tar.gz
tar -xvzf spark-2.4.5-bin-without-hadoop.tgz

mv hadoop-3.2.1 hadoop
mv spark-2.4.5-bin-without-hadoop spark

mkdir hadoop/logs
mkdir dfs
mkdir dfs/data
mkdir dfs/name


rm hadoop-3.2.1.tar.gz
rm spark-2.4.5-bin-without-hadoop.tgz
```

Setup environment:

```bash
nano .bashrc
```

Paste this:

```bash
export JAVA_HOME=`dirname $(dirname $(readlink -f $(which javac)))`
export HADOOP_HOME=/home/vagrant/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export SPARK_HOME=/home/vagrant/spark
export SPARK_LOCAL_IP=192.0.2.20
export PATH=${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${SPARK_HOME}/bin:${SPARK_HOME}/sbin
```

Refresh environment:

```bash
source .bashrc
```

Setup Hadoop:

```bash
nano ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
```

Update this:

```bash
export JAVA_HOME=`dirname $(dirname $(readlink -f $(which javac)))`
export HADOOP_HOME=/home/vagrant/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_LOG_DIR=${HADOOP_HOME}/logs
```

```bash
nano ${HADOOP_HOME}/etc/hadoop/core-site.xml
```

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

```bash
nano ${HADOOP_HOME}/etc/hadoop/hdfs-site.xml
```

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/vagrant/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/vagrant/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

Start Hadoop:

```bash
hdfs namenode -format
${HADOOP_HOME}/sbin/start-dfs.sh
```

Browse Namenode:

[http://192.0.2.20:9870/](http://192.0.2.20:9870/)

Test Hadoop:

```bash
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/vagrant
hdfs dfs -mkdir input
hdfs dfs -put etc/hadoop/*.xml input
hadoop jar hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
hdfs dfs -get output output
cat output/*
hdfs dfs -cat output/*
```

Stop Hadoop:

```bash
${HADOOP_HOME}/sbin/stop-dfs.sh
```

Setup Spark:

```bash
cp ${SPARK_HOME}/conf/spark-env.sh.template spark/conf/spark-env.sh
nano ${SPARK_HOME}/conf/spark-env.sh
```

Paste this:
```bash
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export SPARK_DIST_CLASSPATH=$(${HADOOP_HOME}/bin/hadoop classpath)
```

Setup Yarn:

```bash
nano ${HADOOP_HOME}/etc/hadoop/mapred-site.xml
```

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

```bash
nano ${HADOOP_HOME}/etc/hadoop/yarn-site.xml
```

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

Start Yarn:

```bash
${HADOOP_HOME}/sbin/start-yarn.sh
```

Browse Resource Manager:

[http://192.0.2.20:8088/](http://192.0.2.20:8088/)

Submit Job:

```bash
spark-submit --master yarn --deploy-mode cluster ${SPARK_HOME}/examples/src/main/python/pi.py
```

Stop Yarn:

```bash
${HADOOP_HOME}/sbin/stop-yarn.sh
```







Install Livy:

```bash
curl -O https://downloads.apache.org/incubator/livy/0.7.0-incubating/apache-livy-0.7.0-incubating-bin.zip

unzip apache-livy-0.7.0-incubating-bin.zip

mv apache-livy-0.7.0-incubating-bin livy

rm apache-livy-0.7.0-incubating-bin.zip
```

Start Livy:

```bash
livy/bin/livy-server start
```

Browse Sessions:

[http://192.0.2.20:8998](http://192.0.2.20:8998)

Prepare Job:

```bash
hdfs dfs -mkdir scripts
hdfs dfs -put spark/examples/src/main/python/pi.py scripts
```

Run Job:

```bash
curl --location --request POST 'http://192.0.2.20:8998/batches?doAs=vagrant' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "Py-Pi",
    "file": "/user/vagrant/scripts/pi.py",
    "executorMemory": "1g",
    "executorCores": 1,
    "numExecutors": 1
}'
```
