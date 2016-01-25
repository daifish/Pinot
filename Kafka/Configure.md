##Kafka集群搭建
####使用版本：2.11-0.9.0.0
####下载地址：http://kafka.apache.org/downloads.html  注：我使用的是已有的kafka，下载的话选择接近版本应该没有问题
####参考网址：http://bigcat2013.iteye.com/blog/2175880?utm_source=tuicool&utm_medium=referral
####参考网址：http://www.linuxidc.com/Linux/2014-09/107386p2.htm
####参考网址：http://blog.csdn.net/21aspnet/article/details/19325373
####集群挂掉测试参考网址：http://blog.csdn.net/wangjia184/article/details/37921183

####配置过程：
#####1.安装jdk/jre
#####2.安装zookeeper,可以参考我的笔记 https://github.com/daifish/PinotForYR/blob/master/Zookeeper/Configure.md
#####3.下载解压安装kafka
#####4.修改配置文件/config/server.properties 以下是我的从节点node1的配置（broker.id为1，主节点该值为0）
    ############################ Server Basics #############################
    
    # The id of the broker. This must be set to a unique integer for each broker.
    broker.id=1
    
    ############################# Socket Server Settings #############################
    
    listeners=PLAINTEXT://:9092
    
    # The port the socket server listens on
    #port=9092
    
    # Hostname the broker will bind to. If not set, the server will bind to all interfaces
    host.name=node1
    
    # Hostname the broker will advertise to producers and consumers. If not set, it uses the
    # value for "host.name" if configured.  Otherwise, it will use the value returned from
    # java.net.InetAddress.getCanonicalHostName().
    #advertised.host.name=<hostname routable by clients>
    advertised.host.name=node1
    
    # The port to publish to ZooKeeper for clients to use. If this is not set,
    # it will publish the same port that the broker binds to.
    #advertised.port=<port accessible by clients>
    
    # The number of threads handling network requests
    num.network.threads=3
    
    "server.properties" 125L, 5668C                                                                                                                                    1,1           Top
    # The settings below allow one to configure the flush policy to flush data after a period of time or
    # every N messages (or both). This can be done globally and overridden on a per-topic basis.
    
    # The number of messages to accept before forcing a flush of data to disk
    #log.flush.interval.messages=10000
    
    # The maximum amount of time a message can sit in a log before we force a flush
    #log.flush.interval.ms=1000
    
    ############################# Log Retention Policy #############################
    
    # The following configurations control the disposal of log segments. The policy can
    # be set to delete segments after a period of time, or after a given size has accumulated.
    # A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
    # from the end of the log.
    log.dirs=/opt/kafka_2.11-0.9.0.0/logs
    # The minimum age of a log file to be eligible for deletion
    log.retention.hours=168
    
    # A size-based retention policy for logs. Segments are pruned from the log as long as the remaining
    # segments don't drop below log.retention.bytes.
    #log.retention.bytes=1073741824
    
    # The maximum size of a log segment file. When this size is reached a new log segment will be created.
    log.segment.bytes=1073741824
    
    # The interval at which log segments are checked to see if they can be deleted according
    # to the retention policies
    log.retention.check.interval.ms=300000
    
    # By default the log cleaner is disabled and the log retention policy will default to just delete segments after their retention expires.
    # If log.cleaner.enable=true is set the cleaner will be enabled and individual logs can then be marked for log compaction.
    log.cleaner.enable=false
    
    ############################# Zookeeper #############################
    
    # Zookeeper connection string (see zookeeper docs for details).
    # This is a comma separated host:port pairs, each corresponding to a zk
    # server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
    # You can also append an optional chroot string to the urls to specify the
    # root directory for all kafka znodes.
    zookeeper.connect=ubuntu:2181,node1:2181,node2:2181
    
    # Timeout in ms for connecting to zookeeper
    zookeeper.connection.timeout.ms=6000
#####注意：1.broker.id从0开始，如我的demo中3台机器ubuntu,node1,node2（均为机器名，配置在/etc/hosts里）,该值从0到2
#####2.host.name和advertised.host.name都为当前的机器名，上例中都为node1，其他机器应都为ubuntu,node2
#####3.zookeeper.connect的值为zookeeper集群的值
#####在三台机器上均启动kafka，进入到kafka的目录下执行：
    ./bin/kafka-server-start.sh config/server.properties
#####可以试着创建topic，在一台server上创建producer，另外一台创建consumer，从producer上发送信息，看consumer是否能接收到，以验证集群对否成功
#####创建topic：
    ./bin/kafka-topics.sh -zookeeper ubuntu:2181,node1:2181,node2:2181 -topic test -replication-factor 2 -partitions 5 -create
#####replication-factor表示该topic需要在不同的broker中保存几份，这里replication-factor设置为2, 表示在两个broker中保存
#####查看topic：
    ./bin/kafka-topics.sh -zookeeper ubuntu:2181,node1:2181,node2:2181 -list
    Topic:test	PartitionCount:5	ReplicationFactor:2	Configs:
    	Topic: test	Partition: 0	Leader: 0	Replicas: 0,1	Isr: 0,1
    	Topic: test	Partition: 1	Leader: 1	Replicas: 1,0	Isr: 0,1
    	Topic: test	Partition: 2	Leader: 0	Replicas: 0,1	Isr: 0,1
    	Topic: test	Partition: 3	Leader: 1	Replicas: 1,0	Isr: 0,1
    	Topic: test	Partition: 4	Leader: 0	Replicas: 0,1	Isr: 0,1
#####Leader: 如果有多个brokerBroker保存同一个topic，那么同时只能有一个Broker负责该topic的读写，其它的Broker作为实时备份。负责读写的Broker称为Leader.
#####Replicas : 表示该topic的0分区在0号和1号broker中保存
#####Isr : 表示当前有效的broker, Isr是Replicas的子集    
#####在ubuntu创建producer:
    ./bin/kafka-console-producer.sh -broker-list ubuntu:2181,node1:2181,node2:2181 -topic test
#####在node1创建consumer:
    ./bin/kafka-console-consumer.sh -zookeeper ubuntu:2181,node1:2181,node2:2181 - from-begining -topic test
#####在producer中输入消息，在consumer的终端能显示出相应信息即表示成功                                       
