####Pinot版本:0.016
#####github地址：https://github.com/linkedin/pinot

####Pinot集群的搭建部署：
#####根据github中的介绍，其实主要是根据controller、server、broker启动的数量而定
#####定义三台机器为a,b,c

#####对于测试环境，可以在a启动controller,在b启动server,在c启动broker
#####注意启动操作的时候加上-zkAddress 的值为其中某一台(如a)的zookeeper的地址
#####如-zkAddress 192.168.55.133:2181 一定要保证zookpeeper为同样的值
#####既可以在任意一台机器上进行查询操作
#####同时指定参数-brokerHost 192.168.55.132(c机器的ip) 
#####-brokerPort 8099(启动broker时的默认值，如启动时修改，在本处也选择对应值即可)

#####对于生产环境 如果机器的配置允许，可以保证在每台机器上都启动controller、server、broker
#####保证启动时-zkAddress 的值为相同的值即可

####Pinot项目执行offline过程：
#####1.启动相关的zookeeper、controller、server、broker
#####执行指令：
    bin/pinot-admin.sh StartZookeeper
    bin/pinot-admin.sh StartController 
    bin/pinot-admin.sh StartBroker 
    bin/pinot-admin.sh StartServer
#####本处说明下start指令中可能用到的参数
    bin/pinot-admin.sh StartController -controllerHost host地址 -controllerPort port端口（默认9000） -dataDir 路径（保存数据的文件夹路径，默认在/tmp文件夹下，最好修改） -clusterName 簇名字（默认为PinotCluster） -zkAddress zookeeper地址及端口
    bin/pinot-admin.sh StartBroker -brokerHost host地址 -brokerPort port端口（默认8099） -zkAddress zookeeper地址及端口(默认localhost:2181) -clusterName 簇名字（默认为PinotCluster
    bin/pinot-admin.sh StartServer -serverHost host地址 -serverPort(默认8098) -dataDir 路径（保存server数据的文件夹路径，默认在/tmp文件夹下，最好修改）-segmentDir 路径(保存segments的文件夹路径，同样默认在/tmp下，最好修改) -zkAddress zookeeper地址及端口(默认localhost:2181)
#####2.创建schema 
#####执行指令：
    bin/pinot-admin.sh AddSchema -schemaFile flights-schema.json -exec
#####需要添加-exec 保证执行
#####3.创建table
#####执行指令：
    bin/pinot-admin.sh AddTable -filePath flights-definition.json
#####4.通过hadoop创建索引 
######demo: job.properties
    
    # Segment creation job config:
    path.to.input=/user/pinot/flights-avro
    path.to.output=/user/pinot/flights-segments
    path.to.schema=flights-schema.json
    segment.table.name=flights
    
    # Segment tar push job config:
    push.to.hosts=pinot.example.com
    push.to.port=9000

#####执行创建segment,tarPush操作，即可执行检索操作
    hadoop jar pinot-hadoop-0.016.jar SegmentCreation job.properties 
    hadoop jar pinot-hadoop-0.016.jar SegmentTarPush job.properties 
    bin/pinot-admin.sh PostQuery -query "select count(*) from flights"

####Pinot集群相关工具Helix简单介绍：
#####http://helix.apache.org/0.6.4-docs/tutorial_admin.html
#####在这里 我采用Command Line Interface的 方式

#####Pinot集群默认会使用PinotCluster这个集群，在我们Helix的操作中都默认采用该集群
#####每一个Helix命令操作都需要添加参数 --zkSvr (此处为zookeeper地址，如localhost:2181)

####Helix常用命令：
#####创建cluster:
    ./helix-admin.sh --zkSvr localhost:2181 --addCluster (name)
#####添加node(instance)：
    ./helix-admin.sh --zkSvr localhost:2181 --addNode 簇名字 node名
#####查看cluster信息：
    ./helix-admin.sh --zkSvr localhost:2181 --listClusterInfo PinotCluster(此处代表cluster名字，默认为PinotCluster)
#####查看具体instance(node)信息：
    ./helix-admin.sh --zkSvr localhost:2181 --listInstanceInfo PinotCluster
#####查看cluster中的资源信息:
    ./helix-admin.sh --zkSvr localhost:2181 --listResourceInfo PinotCluster 资源名

#####实际上，当我们startserver, startbroker时，在指定的cluster就会添加相应的server,broker node
#####而当我们添加一个任务的时候，会在指定的cluster添加相应的resource
