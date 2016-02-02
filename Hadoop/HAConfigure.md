##Hadoop2.6.0 + HA 部署

###问题来源：由于hadoop的namenode存在单点故障的问题，在2.0版本之前，hadoop采用secondenamenode作为备用，同步master机的数据。但是由于存在延迟,会造成一定数据量的丢失，为了解决此问题，hadoop在2.x版本提出了HA的策略
####资料参考网站：
####部署配置参考1：http://blog.csdn.net/yczws1/article/details/23566383
####部署配置参考2：http://www.tuicool.com/articles/IvAbUbY
####部署原理参考1: http://www.sizeofvoid.net/hadoop-2-0-namenode-ha-federation-practice-zh/
####部署原理参考2: http://dongxicheng.org/mapreduce-nextgen/hadoop-yarn-ha-in-cdh5/
####部署机器：
#####由于资源有限，采用了3台虚拟机（这种情况较少，很多网络上的文章都给出了至少4台机器的方案，由于集群规模小，数据量有限，但是为了防止主节点的崩溃，还是采用了该方案）具体配置和我之前的配置文件中的机器一样
####部署过程：
#####基本的hadoop集群搭建:可以参考https://github.com/daifish/PinotForYR/blob/master/Hadoop/Configure.md 进行尝试，实际情况中我是基于之前的基础上搭建的
#####zookeeper的搭建:可以参考https://github.com/daifish/PinotForYR/blob/master/Zookeeper/Configure.md

#####下文中主要给出配置文件的配置与解释, 其他的都是很好配的:) 
#####core-site.xml
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop-test</value>
        </property>
        <property>
            <name>io.file.buffer.size</name>
            <value>131072</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/opt/hadoop/data</value>
            <description>A base for other temporary directories.</description>
        </property>
        <property>
            <name>fs.checkpoint.period</name>
            <value>3600</value>
            <description>The number of seconds between two periodic checkpoints.</description>
        </property>
        <property>
            <name>fs.checkpoint.size</name>
            <value>67108864</value>
        </property>
        <property>
            <name>ha.zookeeper.quorum</name>
            <value>ubuntu:2181,node1:2181,node2:2181</value>
        </property>
    </configuration>
#####给出在部署HA时添加的参数的含义
#####fs.defaultFS，这里的值指的是默认的HDFS路径。这里只有一个HDFS集群，在这里指定！该值来自于hdfs-site.xml中的配置（本处我采用了hadoop-test为名字）
#####ha.zookeeper.quorom，嗯，顾名思义，zookeeper集群的配置，和在zookeeper部署时遇到的值一样，用,分隔每台主机及相应端口号即可
#####没介绍的值都为正常部署hadoop时存在的配置
#####hdfs-site.xml
    <configuration>
        <property>
            <name>dfs.nameservices</name>
            <value>hadoop-test</value>
        </property>
        <property>
            <name>dfs.ha.namenodes.hadoop-test</name>
            <value>namenode1,namenode2</value>
        </property>
        <property>
            <name>dfs.namenode.rpc-address.hadoop-test.namenode1</name>
            <value>ubuntu:19000</value>
        </property>
        <property>
            <name>dfs.namenode.rpc-address.hadoop-test.namenode2</name>
            <value>node1:19000</value>
        </property>
        <property>
            <name>dfs.namenode.http-address.hadoop-test.namenode1</name>
            <value>ubuntu:50070</value>
        </property>
        <property>
            <name>dfs.namenode.http-address.hadoop-test.namenode2</name>
            <value>node1:50070</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>3</value>
        </property>
        <property>
            <name>dfs.permissions</name>
            <value>false</value>
        </property>
        <property>
            <name>dfs.blocksize</name>
            <value>268435456</value>
        </property>
        <property>
            <name>dfs.namenode.handler.count</name>
            <value>100</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file://${hadoop.tmp.dir}/dfs/name</value>
        </property>
        <property>
            <name>dfs.namenode.shared.edits.dir</name>
            <value>qjournal://ubuntu:8485;node1:8485;node2:8485/hadoop-test</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file://${hadoop.tmp.dir}/dfs/data</value>
        </property>
        <property>
            <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
            <value>false</value>
        </property>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>node1:50090</value>
        </property>
        <property>
            <name>dfs.journalnode.edits.dir</name>
            <value>/opt/hadoop/data/journal</value>
        </property>
        <property>
            <name>dfs.ha.fencing.methods</name>
            <value>sshfence</value>
        </property>
        <property>
            <name>dfs.ha.fencing.ssh.private-key-files</name>
            <value>/home/www/.ssh/id_dsa</value>
        </property>
        <property>
            <name>dfs.ha.fencing.ssh.connect-timeout</name>
            <value>10000</value>
        </property>
        <property>
            <name>dfs.ha.automatic-failover.enabled.hadoop-test</name>
            <value>true</value>
        </property>
        <property>
            <name>dfs.client.failover.proxy.provider.hadoop-test</name>
            <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
        </property>
    </configuration>
#####给出在部署HA时添加的参数的含义，好吧，正如你看到的，这个配置文件好长 - -! take it easy~ :)
#####dfs.nameservices，这个值就对应core-site.xml中的fs.defaultFS的值
#####dfs.ha.namenodes.hadoop-test, 这里要注意，正常而言这个property名字就是dfs.ha.namenodes.{nameservicesname},我们这里用dfs.nameservices的值hadoop-test代替那个值。 对于这个property，他的value则是你要选择作为namenode的节点，名字嘛，随便起，不重名就ok啦～～:)
#####dfs.namenode.rpc-address.hadoop-test.namenode1/2, property名字和上一个变量是一个意思，我们要对hadoop-test中的namenode1配置，它的值表示的是指定ubuntu机器的RPC地址（这个概念需要理解一下，暂时不了解）
#####dfs.namenode.http-address.hadoop-test.namenode1/2, 同样的值，这个property的值表示ubuntu机器的http地址
#####dfs.namenode.shared.edits.dir, 指定hadoop-test的两个NameNode共享edits文件目录时，使用的JournalNode集群信息,值得注意的是，这个是以;分隔的，而不是,
#####dfs.journalnode.edits.dir, 指定JournalNode集群在对NameNode的目录进行共享时，自己存储数据的磁盘路径。journal是启动journalnode自动生成
#####dfs.ha.fencing.methods, 一旦需要NameNode切换，使用ssh方式进行操作
#####dfs.ha.fencing.ssh.private-key-files,使用ssh时用的密钥存储的位置,遇到个坑在这里，你本地一般都是~/.ssh/id_dsa(对linux了解的还是不够),实际上在这里写要用/home/username表示~，如果这里写错了，修改后每次要杀掉zookeeper的zkfc并重启，下文会介绍，否则没有效果
#####dfs.ha.fencing.ssh.connect-timeout, 很明显，超时时间,不是么～～
#####dfs.ha.automatic-failover.enabled.hadoop-test, 很关键，这个值为true的话，master挂掉会自动将standby变为active的，但是目测不能实现命令行手动切换（手动切换要求该值为false）
#####dfs.client.failover.proxy.provider.hadoop-test, 自动切换时使用的代理类
#####其实并不多，不是么:)，剩下的配置文件基本没变动，接下来启动起来吧～～
#####启动zookeeper,在自己的zookeeper路径下:
    /opt/zookeeper/bin/zkServer.sh start
#####格式化zookeeper集群：
    /opt/zookeeper/bin/zkCli.sh
    ls / 
#####你能看到当前zookeeper的znode节点情况，来到hadoop的目录执行
    /opt/hadoop/bin/hdfs zkfc –formatZK
#####这时候再查看zookeeper集群的znode节点情况会有hadoop-ha这个值，嗯，那就ok了，继续吧～
#####启动JournalNode集群，在所有的虚拟机上运行:
    /opt/hadoop/sbin/hadoop-daemon.sh start journalnode
#####jps之后：
    23396 JournalNode    （journalnode）
    23598 Jps    
    22491 QuorumPeerMain  (zookeeper)
#####格式化namenode：
#####在master机ubuntu上运行
    /opt/hadoop/bin/hdfs namenode -format
#####或者任意一台机器上运行
    /opt/hadoopbin/hdfs namenode -format -clusterId namenode
#####这里有个坑，可能会出现Incompatible clusterIDs的错误，这是因为在执行“hdfs namenode -format”之前，没有清空DataNode节点的data目录，所以有效的解决措施就是清空所有DataNode的data目录，但注意不要将data目录本身给删除了。
#####data目录由core-site.xml文件中的属性“dfs.datanode.data.dir”指定 参考网址：http://www.iyunv.com/thread-18061-1-1.html
#####启动格式化过的那个namenode(我们默认为ubuntu机器)
    /opt/hadoop/sbin/hadoop-daemon.sh start namenode
#####把namenode1的数据同步到namenode2中(实际机器也就是从master到node1)，在node1机器上执行
    /opt/hadoop/bin/hdfs namenode –bootstrapStandby
#####启动namenode2(node1机器)的namenode
    /opt/hadoop/sbin/hadoop-daemon.sh start namenode
#####现在看192.168.55.133/192.168.55.128:50070(嗯，我的ubuntu主机和node1主机)两台机器可以看到都有namenode节点了，不过都是standby
#####启动datanode,yarn,我直接在ubuntu（我的master机器上）start-all了
    /opt/hadoop/sbin/start-all.sh
#####在namenode机器上启动ZooKeeperFailoverCotroller，我这里就是ubuntu和node1两台机器
    /opt/hadoop/sbin/hadoop-daemon.sh start zkfc
#####这个时候一台机器就是active，而另一台就是standby了，讲道理而言，这个先active的是先启动的机器（有待验证）
#####jps一下，看看有多少进程了
    24599 DFSZKFailoverController    (zkfc)  
    23396 JournalNode      
    24232 DataNode      
    23558 NameNode      
    22491 QuorumPeerMain      
    24654 Jps   
#####验证HA的故障自动转移是否好用 首先在当前active机器杀掉namenode，比如
    kill -9 23558
#####再打开网页看看是否namenode已经转变了，之前standby的机器变为active，而死掉的机器重启服务后变为standby，嗯，恭喜你，成功了～ :)


