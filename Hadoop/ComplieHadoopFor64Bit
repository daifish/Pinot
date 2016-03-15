####编译hadoop 2.6.4 64位记录

#####版本：hadoop 2.6.4
#####环境：ubuntu 14.04 maven 3.3.9 java 1.8

#####原因：官网下载的hadoop 2.x系列都是32位的，当用在64位linux系统上的时候会出现问题，需要自己手动编译
#####记录工作流程：
#####1.首先到hadoop的官网下载2.6.4的源代码
#####2.将下载好的包传到linux服务器上
#####3.首先解决pom.xml的bug：hadoop2.x存在的bug，在解压好的hadoop文件路径下打开pom.xml
    vim hadoop-common-project/hadoop-auth/pom.xml
    #在<dependencys></dependencys>中插入以下内容
    <dependency>
      <groupId>org.mortbay.jetty</groupId>
      <artifactId>jetty-util</artifactId>
      <scope>test</scope>
    </dependency> 
#####4.安装protoc：
    sudo apt-get update //更新软件源
    sudo apt-get install build-essential //安装c编译器，此时要翻墙
    #安装protoc
    wget https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz
    tar -xzf protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0
    ./configure
    make
    make check
    make install
    #验证
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    protoc --version
    libprotoc 2.5.0 //成功
#####5.安装cmake
    sudo apt-get install cmake
#####6.安装libglib2.0-dev
    sudo apt-get install libglib2.0-dev
#####7.安装libssl-dev
    sudo apt-get install libssl-dev
#####8.编译:
    cd hadoop-2.6.4-src
    //下面的指令应该是适合低版本的java，如1.6 1.7 我使用的1.8执行下面的语句报错
    mvn package -Pdist,native -DskipTests -Dtar
#####9.遇到的错误：
#####9.1. 用java1.8执行8中的mvn指令报错，请执行下面的语句
    mvn clean package -Pdist -Dtar -Dmaven.javadoc.skip=true -DskipTests -fail-at-end -Pnative
#####9.2. sudo apt-get install xxx时报
    E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
    E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it
    #执行下面的指令即可
    sudo rm /var/cache/apt/archives/lock
    sudo rm /var/lib/dpkg/lock
#####10.查看编译成果：都保存在hadoop-2.6.4-src/hadoop-dist/target/目录中
    cd /opt/hadoop-2.6.4-src/hadoop-dist/target/hadoop-2.6.4/lib/native/
    file libhadoop.so.1.0.0
    #出现以下即可，libhadoop.so.1.0.0为64位
    libhadoop.so.1.0.0: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=fb43b4ebd092ae8b4a427719b8907e6fdb223ed9, not stripped
#####11.编译结果：hadoop-2.6.4-src/hadoop-dist/target/hadoop-2.6.4为编译后的文件夹，hadoop-2.6.4-src/hadoop-dist/target/hadoop-2.6.4.tar.gz为编译后的打包文件
