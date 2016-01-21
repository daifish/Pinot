####Hbase使用版本:0.98.11
####虚拟机:Ubuntu14.04
####参考配置链接:http://www.aboutyun.com/thread-7746-1-1.html
#####注:上方配置链接使用版本为0.96，基本适用0.98

####配置Hbase前需要配置好Hadoop,而且Hadoop和HBbase的版本存在对应关系，注意版本之间的适配

####值得注意、经常出现的问题的原因很有可能是集群中机器的时间差距过大造成的，相关解决办法如下：
#####NTP：集群的时钟要保证基本的一致。稍有不一致是可以容忍的，但是很大的不一致会 造成奇怪的行为。 
######可以选择运行 NTP 或者其他什么东西来同步你的时间.
######在本处 我使用date -s "当前时间" 进行同一时间操作，保证集群的机器之间的时间差不要太大，这个时间差在配置文件中会有涉及

####配置信息：
#####在Hbase的conf文件夹下配置具体信息，我的配置如下
hbase-site.xml

    <configuration>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
      #是否为分布式
    </property>
    <property>
      <name>hbase.rootdir</name>
      <value>hdfs://ubuntu:19000/hbase</value>
    </property>
    <property>
        <name>hbase.master.maxclockskew</name>
        <value>150000</value>
        #机器间的时间差，即上文中我们提到的
    </property>
    <property>
        <name>hbase.tmp.dir</name>
        <value>file:/opt/hbase/tmp</value>
        #临时文件的文件夹
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>file:/opt/hbase/zookeeper</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>ubuntu,node1,node2</value>
        #此处很关键，zookeeper管理的节点，以,分隔，本处我使用3个节点，就都记录在此
    </property>
    </configuration>

regionservers

    node1
    node2
#####两个从节点，列在这里的server会随着集群的启动而启动，集群的停止而停止.

#####启动Hbase:在Hbase的当前级别文件夹,执行bin/start-hbase.sh即可
#####终止Hbase:在Hbase的当前级别文件夹,执行bin/stop-hbase.sh即可

