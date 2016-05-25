####WordNet介绍：http://wordnet.princeton.edu
####使用版本：3.0
####下载地址：官网 or http://conll.cemantix.org/2011/download/WordNet-3.0.tar.gz

####安装过程：
####1.下载tar.gz包并解压 本次安装到了/opt目录下
####2.安装tcl、tk
    sudo apt-get install tcl8.4-dev
    sudo apt-get install tk8.4-dev
    #默认安装到/usr/lib处
####3.安装xinit
    sudo apt-get install xinit
####4. configure , make, make install
    cd /opt/Word-net3.0
    ./configure --with-tk=/usr/lib/tk8.4/ --with-tcl=/usr/lib/tcl8.4
    make
    make install
####5. 依次执行以下指令
    export DISPLAY=:0
    sudo su
    xhost +
    startx
    su www(当前用户为www)
    在/opt/Word-net3.0/目录下执行wnb进入图形化界面
####此外，可以在/opt/Word-net3.0/目录下 执行wn 指令 进行操作，只是该操作并不是图形化界面操作的. 操作格式 wn 单词 [options] [searchtype]
