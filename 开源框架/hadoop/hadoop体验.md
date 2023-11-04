# hadoop



```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.aarch64


http://10.211.55.15:9870/


exec /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.aarch64/bin/java -Dproc_jar -Djava.net.preferIPv4Stack=true -Dyarn.log.dir=/home/parallels/hadoop-3.3.6/logs -Dyarn.log.file=hadoop.log -Dyarn.home.dir=/home/parallels/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/home/parallels/hadoop-3.3.6/lib/native -Dhadoop.log.dir=/home/parallels/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop.log -Dhadoop.home.dir=/home/parallels/hadoop-3.3.6 -Dhadoop.id.str=parallels -Dhadoop.root.logger=INFO,console -Dhadoop.policy.file=hadoop-policy.xml -Dhadoop.security.logger=INFO,NullAppender org.apache.hadoop.util.RunJar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar grep input output 'dfs[a-z.]+'


-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5555


file:/tmp/hadoop/mapred/staging/parallels152299599/.staging


paralle+ 3090787       1  9 22:25 ?        00:00:02 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.aarch64/bin/java -Dproc_namenode -Djava.net.preferIPv4Stack=true -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dyarn.log.dir=/home/parallels/hadoop-3.3.6/logs -Dyarn.log.file=hadoop-parallels-namenode-fedora.log -Dyarn.home.dir=/home/parallels/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/home/parallels/hadoop-3.3.6/lib/native -Dhadoop.log.dir=/home/parallels/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop-parallels-namenode-fedora.log -Dhadoop.home.dir=/home/parallels/hadoop-3.3.6 -Dhadoop.id.str=parallels -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.namenode.NameNode
paralle+ 3090993       1  9 22:25 ?        00:00:02 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.aarch64/bin/java -Dproc_datanode -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=ERROR,RFAS -Dyarn.log.dir=/home/parallels/hadoop-3.3.6/logs -Dyarn.log.file=hadoop-parallels-datanode-fedora.log -Dyarn.home.dir=/home/parallels/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/home/parallels/hadoop-3.3.6/lib/native -Dhadoop.log.dir=/home/parallels/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop-parallels-datanode-fedora.log -Dhadoop.home.dir=/home/parallels/hadoop-3.3.6 -Dhadoop.id.str=parallels -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.datanode.DataNode
paralle+ 3091223       1  7 22:25 ?        00:00:01 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.362.b09-1.fc36.aarch64/bin/java -Dproc_secondarynamenode -Djava.net.preferIPv4Stack=true -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dyarn.log.dir=/home/parallels/hadoop-3.3.6/logs -Dyarn.log.file=hadoop-parallels-secondarynamenode-fedora.log -Dyarn.home.dir=/home/parallels/hadoop-3.3.6 -Dyarn.root.logger=INFO,console -Djava.library.path=/home/parallels/hadoop-3.3.6/lib/native -Dhadoop.log.dir=/home/parallels/hadoop-3.3.6/logs -Dhadoop.log.file=hadoop-parallels-secondarynamenode-fedora.log -Dhadoop.home.dir=/home/parallels/hadoop-3.3.6 -Dhadoop.id.str=parallels -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
paralle+ 3091423 3086998  0 22:25 pts/5    00:00:00 grep --color=auto java




```

