##Zookeeper集群搭建
####Zookeeper版本：3.4.6
####下载地址：http://apache.opencas.org/zookeeper/stable/zookeeper-3.4.6.tar.gz 
####参考链接：http://bigcat2013.iteye.com/blog/2175538

####安装步骤：
#####1.安装jdk/jre
#####2.下载并解压zookeeper，地址如上
#####3.配置conf
#####使用“cp zoo_sample.cfg  zoo.cfg”来创建一个zookeeper配置文件，在zoo.cfg中配置syncLimit,dataDir，clientPort,autopurge.purgeInterval,以及集群的server list
#####配置如下：
    # The number of milliseconds of each tick
    tickTime=2000
    # The number of ticks that the initial
    # synchronization phase can take
    initLimit=10
    # The number of ticks that can pass between
    # sending a request and getting an acknowledgement
    syncLimit=5
    # the directory where the snapshot is stored.
    # do not use /tmp for storage, /tmp here is just
    # example sakes.
    dataDir=/opt/zookeeper-3.4.6/tmp/zookeeper
    # the port at which the clients will connect
    clientPort=2181
    # the maximum number of client connections.
    # increase this if you need to handle more clients
    #maxClientCnxns=60
    #
    # Be sure to read the maintenance section of the
    # administrator guide before turning on autopurge.
    #
    # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
    #
    # The number of snapshots to retain in dataDir
    #autopurge.snapRetainCount=3
    # Purge task interval in hours
    # Set to "0" to disable auto purge feature
    autopurge.purgeInterval=1
    server.1=ubuntu:2888:3888
    server.2=node1:2888:3888
    server.3=node2:2888:3888
#####注：本处我使用了3台机器，名字分别为ubuntu,node1,node2 这些配置都在/etc/hosts中配置ip 和 主机名，此外，要创建dataDir对应的文件夹，该文件夹原则上与其他部署的应用的文件夹在同一级文件夹中，方便管理
#####通过“scp -r ”把配置好的zookeeper目录copy到其他两台server上：
#####在dataDir(比如我这里就是/opt/zookeeper-3.4.6/tmp/zookeeper)中创建myid文件：
    sudo vim myid
#####内容即对应为1，2，3 即主机的id号
#####分别启动三台zookeeper，并检查集群状态：
#####使用
    sudo ./bin/zkServer.sh start
#####启动zookeeper
#####使用
    sudo ./bin/zkServer.sh status
#####检查集群状态
#####操作的效果如下：
    www@ubuntu:/opt/zookeeper-3.4.6/bin$ ./zkServer.sh start
    JMX enabled by default
    Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
    www@ubuntu:/opt/zookeeper-3.4.6/bin$ ./zkServer.sh status
    JMX enabled by default
    Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
    Mode: follower
#####mode显示了所在server在集群中所扮演的角色，每个server的角色不是固定的，leader是通过zookeeper的Fast Leader 选举算法产生，三台zookeeper集群就这么搭建好了，大家可以根据自己实际的项目需要再做一些详细的配置。
