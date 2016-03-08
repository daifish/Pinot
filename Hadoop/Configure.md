####Hadoop使用版本:2.6.0
####Hadoop配置方式:全分布式
####虚拟机:Ubuntu 14.04
####参考官网配置:http://hadoop.apache.org/docs/r2.6.0/

#####!!!保证集群三台机器的配置一样
#####我使用的vmvare 创建了一个虚拟机，剩下的两个则为克隆第一个虚拟机
#####配置3台机器，以我的配置为例
        /etc/hosts:
        192.168.55.133 ubuntu   #主机
        192.168.55.128 node1    #从节点
        192.168.55.132 node2    #从节点
#####在/etc/hostname将host名依次改好，主机为ubuntu 从节点分别为node1, node2

#####接下来安装jdk 下载hadoop相应版本
#####新建用户，在这里我新建www用户、www用户组（分配root权限，还可以在配置文件里更改，这里用命令的方式）
        sudo groupadd hadoop    //设置hadoop用户组
        sudo useradd –s /bin/bash –d /home/www –m www –g www  –G admin   //添加一个www用户，此用户属于www用户组，且具有admin权限。
        sudo passwd www   //设置用户zhm登录密码
        su www   //切换到zhm用户中

#####安装ssh
        sudo apt-get install ssh
#####配置多机间的无密码登录（ssh的原理请自行查阅,本处只着重配置）在主机执行以下命令
        ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa //产生公私密钥，产生在用户主目录下的.ssh目录中 即~/.ssh
        cd ~/.ssh 
        ls //可以看到id_dsa（私钥） id_dsa.pub（公钥） 两个文件
        cat id_das.pub >> authorized_keys //将公钥文件复制成authorized_keys文件
        #在两个从节点~/.ssh 目录执行命令
        scp www@master:~/ssh/id_dsa.pub ./master_dsa.pub //需要输入master机的密码
        cat master_dsa.pub >> authorized_keys
        #在master节点 执行命令
        ssh node1 //进入到node1节点即为成功，首次连接需要人工询问，之后无需人工询问
        exit //退出node1
        ssh node2
        exit
        

#####贴一下我的配置 hadoop安装在/opt/hadoop

#####/opt/hadoop/etc/hadoop/core-site.xml:

        <configuration>
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://ubuntu:19000</value>
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
        </configuration>

#####/opt/hadoop/etc/hadoop/hdfs-site.xml:

        <configuration>
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
                <name>dfs.datanode.data.dir</name>
                <value>file://${hadoop.tmp.dir}/dfs/data</value>
            </property>
            <property>
                <name>dfs.namenode.datanode.registration.ip-hostname-check</name>
                <value>false</value>
            </property>
        </configuration>
        
#####/opt/hadoop/etc/hadoop/yarn-site.xml:
        
        <configuration>
        <!-- Site specific YARN configuration properties -->
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
             <name>yarn.resourcemanager.resource-tracker.address</name>
             <value>ubuntu:8031</value>
        </property>
        <property>
            <name>yarn.resourcemanager.address</name>
            <value>ubuntu:8032</value>
          </property>
          <property>
            <name>yarn.resourcemanager.admin.address</name>
            <value>ubuntu:8033</value>
          </property>
        </configuration>

#####/opt/hadoop/etc/hadoop/slaves:

        node1
        node2

#####配置之后sbin/start-all.sh启动hadoop项目
#####jps主节点ubuntu共4个进程：jps,namenode,secondarynamenode,resourcemanager
#####jps从节点node1(node2)共3个进程：jps,datanode,nodemanager
#####浏览器端打开主节点：50070查看 如果2个datanode即ok 测试url: http://192.168.55.133:50070

####遇到的bug及问题：
#####由于是直接克隆第一台虚拟机，而第一台虚拟机已经进行了format操作，可能会引起查看50070端口时只有1个live node
####采用的解决方案：
#####将data目录中的数据都删除，重新format 核心的问题即data/dfs/data/current/VERSION中的storeID冲突
#####参考网址：http://www.jiancool.com/article/723166056/;jsessionid=041DAC904C25B1F1C6A249A5C76FFED4



