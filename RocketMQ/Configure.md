####版本：3.2.6
####配置环境：3台虚拟机，ubuntu14.04版本，单核2g内存
####快速开始：下载rocketmq 下载链接： https://github.com/alibaba/RocketMQ
####部署过程：采用单namesrv,单master，配2个slave
#####1.安装jdk
#####2.解压rocketmq的tar.gz文件，进入安装目录，如
    /opt/alibaba-rocketmq
#####3.在a机器启动nameserver（当然可以专门拿出一个或多个机器启动nameserver，只需在接下来爹启动broker时指定好nameserver的ip:port;ip:port即可）
    cd /opt/alibaba-rocketmq/bin
    sh mqnamesrv //默认启动9876端口
#####jps 查看进程如下即ok：
    2045 NamesrvStartup 
#####默认会分配4g内存，修改的地方在：
    /opt/alibaba-rocketmq/bin/runserver.sh 中的JAVA_OPT参数，试验中我的参数如下
    JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=320m"
#####4.在a机器启动master
    cd /opt/alibaba-rocketmq/bin
    sh mqbroker -n "192.168.55.133:9876" -c ../conf/2m-2s-sync/broker-b.properties // -n 指定namesrv -c 指定启动broker的参数文件的位置（注意我这里用的是broker-b.properties，这是下载后自带的，并没用-a.propertis）
#####broker-b.propertis配置如下：
    brokerClusterName=DefaultCluster
    brokerName=broker-b //根据该属性进行master-slave配对
    brokerId=0 //master id必须为0，多slave情况下根据id区分
    listenPort=10911 //监听端口，可以显示更改，启动后为192.168.55.133:10911
    storePathRootDir=/opt/alibaba-rocketmq/data/store  
    storePathCommitLog=/opt/alibaba-rocketmq/data/store/commitlog //日志文件的目录，可自定义
    deleteWhen=04
    fileReservedTime=48
    brokerRole=SYNC_MASTER //主机身份
    flushDiskType=ASYNC_FLUSH
#####注意，启动master和slave都是启动一个broker,如果想在每台机器上开一个master，slave就要修改listenport（本例子中是a机器启动master,b、c机器启动slave），此外，启动broker的默认内存同样为4g，修改处在
    /opt/alibaba-rocketmq/bin/runbroker.sh 中的JAVA_OPT参数，试验中我的参数如下
    JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=320m"
#####5.在b机器、c机器分别启动slave
    cd /opt/alibaba-rocketmq/bin
    sh mqbroker -n "192.168.55.133:9876" -c ../conf/2m-2s-sync/broker-b-s.properties
#####broker-b-s.properties配置如下：
    brokerClusterName=DefaultCluster
    brokerName=broker-b //对应master机的brokername
    brokerId=1 //slave id不为0， 如有两个slave可分别为1 和 2
    listenPort=10911 //监听端口，可以显示更改，启动后为192.168.55.132:10911
    storePathRootDir=/opt/alibaba-rocketmq/data/store  
    storePathCommitLog=/opt/alibaba-rocketmq/data/store/commitlog //日志文件的目录，可自定义
    deleteWhen=04
    fileReservedTime=48
    brokerRole=SLAVE //slave身份
    flushDiskType=ASYNC_FLUSH
#####启动后的jps出现如下即ok：
    1641 BrokerStartup
#####注意，如果启动broker(无论是master,slave)都可以在启动-n 参数中输入多个namesrv地址，如-n "192.168.55.132:9876;192.168.55.128:9876"
#####性能测试参考：http://blog.csdn.net/dhdhdh0920/article/details/42710787
#####本机性能测试：单线程发送intps 500左右 
