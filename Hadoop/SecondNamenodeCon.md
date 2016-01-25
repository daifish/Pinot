##SecondNamenode的配置与设置：
####参考资料：http://www.aboutyun.com/thread-8146-1-1.html
####Hadoop版本：2.6.0
####环境配置:3台虚拟机，机器名为ubuntu, node1, node2

####配置过程:
#####在hadoop/etc/hadoop目录下新建masters文件，添加作为secondeNamenode的节点
#####我的masters文件里的内容如下：
    node1
#####即指定node1机器上运行secondeNamenode
#####注：如果你想单独配置一台机器，那么在这个文件里面，填写这个节点的ip地址或则是hostname，如果是多台，则在masters里面写上多个，一行一个，我们这里指定一个
#####在hdfs-site.xml的configure中添加property如下：
    <property>
      <name>dfs.namenode.secondary.http-address</name>
      <value>slave1:50090</value>
    </property>
#####在core-site.xml的configure中添加property如下：
    <property>
      <name>fs.checkpoint.period</name>
      <value>3600</value>
      <description>The number of seconds between two periodic checkpoints.
      </description>
    </property>
    <property>
      <name>fs.checkpoint.size</name>
      <value>67108864</value>
    </property>
#####在主节点启动hadoop任务之后，可以在控制台看到类似输出：
    Starting namenodes on [ubuntu]
    ubuntu: starting namenode, logging to /opt/hadoop/logs/hadoop-www-namenode-ubuntu.out
    node2: starting datanode, logging to /opt/hadoop/logs/hadoop-www-datanode-node2.out
    node1: starting datanode, logging to /opt/hadoop/logs/hadoop-www-datanode-node1.out
    Starting secondary namenodes [node1]
    node1: starting secondarynamenode, logging to /opt/hadoop/logs/hadoop-www-secondarynamenode-node1.out
    Safe mode is OFF
    starting yarn daemons
    starting resourcemanager, logging to /opt/hadoop/logs/yarn-www-resourcemanager-ubuntu.out
    node2: starting nodemanager, logging to /opt/hadoop/logs/yarn-www-nodemanager-node2.out
    node1: starting nodemanager, logging to /opt/hadoop/logs/yarn-www-nodemanager-node1.out
    node2: starting zookeeper, logging to /opt/hbase/bin/../logs/hbase-www-zookeeper-node2.out
    node1: starting zookeeper, logging to /opt/hbase/bin/../logs/hbase-www-zookeeper-node1.out
    ubuntu: starting zookeeper, logging to /opt/hbase/bin/../logs/hbase-www-zookeeper-ubuntu.out
    starting master, logging to /opt/hbase/bin/../logs/hbase-www-master-ubuntu.out
    www@ubuntu:/opt$ node2: starting regionserver, logging to /opt/hbase/bin/../logs/hbase-www-regionserver-node2.out
    node1: starting regionserver, logging to /opt/hbase/bin/../logs/hbase-www-regionserver-node1.out
#####在node1节点输入jps，查看进程可看到如下：
    1792 Jps
    1360 SecondaryNameNode
    1633 HQuorumPeer
    1270 DataNode
    1723 HRegionServer
    1455 NodeManager
#####有以上进程表示ok，当然，有一部分是zookeeper（1633，1723）, 核心是secondarynamenode的启动
