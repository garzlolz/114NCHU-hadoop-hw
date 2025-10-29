# 雲端運算作業

## 第三章

1. 安裝 guestaddiition
安裝玩 ubuntu 後開啟 guset addition 會報錯 缺少 bzipz
開啟 terminal 輸入 sudo apt update
接著跑 sudo apt install bzip2 tar
就可以安裝了
![更新基本套件](./screen%20shot/VirtualBox_master_29_10_2025_19_51_04.png)

2. 接著關機，設定雙向剪貼簿
![雙向剪貼簿](./screen%20shot/Snipaste_2025-10-29_19-52-50.png)
3. 先檢查 python 版本再考慮 spark 跟 hadoop 版本接著安裝 Java
建議版本
python3 -v => Python 3.12.3

## 第四章

安裝 java

java -V => sudo apt install openjdk-17-jre-headless
![檢查版本](./screen%20shot/VirtualBox_master_29_10_2025_19_54_59.png)

安裝 spark

檢查 java 安裝位置 (目的是為了 bashrc使用)
hduser@hadoop:~$ update-alternatives --display java

接著要安裝ssh
先安裝 ssh => sudo apt-get install ssh

接下來安裝 rsync
sudo apt-get install ssh

接下來設定 ssh
在ubuntu 24.04 dsa 已經被棄用，產生 rsa 金鑰 => hduser@hadoop:~$ ssh-keygen -t rsa -f ~/.ssh/id_rsa
檢查 rsa => ll ~/.ssh
將產生的金鑰放到 auth keys =>
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

安裝 hadoop，檢查後 jdk 17 支援 3.3.6
下載 hadoop => wget <https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz>
解壓縮 hadoop.tar => sudo tar -zxvf hadoop-3.3.6.tar.gz
搬移 hadoop folder => sudo mv hadoop-3.3.6 /usr/local/hadoop
檢查 hadoop => ll /usr/local/hadoop

安裝 gedit
sudo apt install gedit、sudo apt install gedit-plugins

編輯 ~/.bashrc
sudo gedit ~/.bashrc
打開後拉到最底下  輸入

# ================================================================= # HADOOOP & JAVA ENVIRONMENT VARIABLES (FIXED FOR JDK 17) # ================================================================= # 1. JAVA HOME 設定 (已修正為 JDK 17 的正確路徑) export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 # 2. HADOOP HOME 設定 (您的路徑是正確的) export HADOOP_HOME=/usr/local/hadoop # 3. 將 Java 和 Hadoop 的 bin/sbin 目錄加入 PATH export PATH=$PATH:$JAVA_HOME/bin export PATH=$PATH:$HADOOP_HOME/bin export PATH=$PATH:$HADOOP_HOME/sbin # 4. 設定各別元件的 HOME 變數 (保持與 HADOOP_HOME 相同) export HADOOP_MAPRED_HOME=$HADOOP_HOME export HADOOP_COMMON_HOME=$HADOOP_HOME export HADOOP_HDFS_HOME=$HADOOP_HOME export YARN_HOME=$HADOOP_HOME # 5. Native Library 和 HADOOP_OPTS export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib" export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH

=> 儲存後reload bashrc => source ~/.bashrc

修改 hadoop 組態設定檔
sudo gedit /usr/local/hadoop/etc/hadoop/hadoop-env.sh
search JAVA_HOME 修改目錄為 => export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

修改 core.site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/core-site.xm
config 新增 prop
 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
 </property>

設定 yarn-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/yarn-site.xml
新增prop
 <property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
 </property>
 <property>
  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
 </property>

設定 mapred.site.xml => gedit /usr/local/hadoop/etc/hadoop/mapred-site.xml
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

設定 hdfs-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
    </property>

建立與格式化HDFS目錄
建立 namenode 資料儲存目錄
sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/namenode
建立 datanode 資料儲存目錄
sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode
更改目錄擁有者
sudo chown hduser:hduser -R /usr/local/hadoop
將HDFS 格式化
hadoop namenode -format
放寬 Java 權限
gedit /usr/local/hadoop/etc/hadoop/hadoop-env.sh
加入

解決 Java 9+ 的模組化安全錯誤

export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.lang=ALL-UNNAMED"
export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.nio=ALL-UNNAMED"

因為 新版的 hadoop Hdfs 的 Web 在 <http://localhost:9870/>
要改為與課本相同的 <http://localhost:50070>
gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
新增 prop
    <property>
     <name>dfs.namenode.http-address</name>
     <value>0.0.0.0:50070</value>
     <description>The address and port that the NameNode web UI will listen on.</description>
    </property>

啟動 hadoop
start-all.sh

## 第五、六章

關閉虛擬機，再製後編輯網路介面卡2，啟用網路並開啟內部網路
啟動後編輯網路設定黨
新版 ubuntu 不是用  sudo gedit /etc/network/interfaces  而是用 Netplan
且新版 ubuntu 已經棄用 eth0、eth1 這種網卡命名
  sudo gedit /etc/netplan/01-network-manager-all.yaml 編輯
network:
  version: 2
  renderer: networkd
  ethernets:
    # 網卡 1: 用於對外連網 (NAT)。必須使用 DHCP 來獲取正確的 Gateway (10.0.2.2)。
    enp0s3:
      dhcp4: true
      nameservers:
        # 強制指定 Google DNS，以避免 DHCP 提供的 DNS 有問題。
        addresses: [8.8.8.8, 8.8.4.4]

    # 網卡 2: 用於集群內部通訊 (Host-Only)。只需靜態 IP。
    enp0s8:
      dhcp4: no
      addresses: [192.168.56.101/24]

 更改權限 => sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
apply => sudo netplan apply
驗證 => hduser@hadoop:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c3:71:52 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 metric 100 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86398sec preferred_lft 86398sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e3:5f:8a brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever

設定 data1 hostname => sudo hostnamectl set-hostname data1

開啟編輯 hosts => sudo gedit /etc/hosts
底部新增

Masternode and Worker nodes for Hadoop Cluster

192.168.56.100 master
192.168.56.101 data1
192.168.56.102 data2
192.168.56.103 data3

編輯 core-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/core-site.xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
 </property>
改為
<property>
  <name>fs.default.name</name>
  <value>hdfs://master:9000</value>
 </property>

因為 yarn-site.xml 是 Mapredice2(Yarn) ，編輯 yarn-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/yarn-site.xml
新增 prop
<configuration>
<!-- Site specific YARN configuration properties -->
 <property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
 </property>
 <property>
  <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
  <value>org.apache.hadoop.mapred.ShuffleHandler</value>
 </property>
 <property>
      <name>yarn.resourcemanager.resource-tracker.address</name>
      <value>master:8025</value>
 </property>
 <property>
      <name>yarn.resourcemanager.scheduler.address</name>
      <value>master:8030</value>
 </property>
 <property>
      <name>yarn.resourcemanager.address</name>
      <value>master:8050</value>
 </property>
</configuration>

編輯mapred-site.xml => sudo gedit /usr/local/hadoop/etc/hadoop/mapred-site.xml
<configuration>
 <property>
  <name>mapred.job.tracker</name>
  <value>master:54311</value>
 </property>
</configuration>

編輯 hdfs-site.xml => sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/datanode</value>
    </property>
    <property>
     <name>dfs.namenode.http-address</name>
     <value>0.0.0.0:50070</value>
     <description>The address and port that the NameNode web UI will listen on.</description>
    </property>
</configuration>
重啟data 1
更新 net-tools => sudo apt install net-tools

複製 data1 to data2、3、master
hduser@data1:~$ sudo hostnamectl set-hostname data2、3、master
hduser@data1:~$ sudo gedit /etc/netplan/01-network-manager-all.yaml
修改 ip 102、103、100
hduser@data1:~$ sudo netplan apply
hduser@data1:~$ ip a
檢查 => hduser@data1:~$ sudo gedit /etc/hostname
檢查網路 => ifconfig

啟動並 修改master node
設定 hdfs-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
     <name>dfs.datanode.data.dir</name>
     <value>file:///usr/local/hadoop/hadoop_data/hdfs/datanode</value>
    <description>DataNode directory for data storage.</description>
</property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/hadoop_data/hdfs/namenode</value>
    </property>
    <property>
     <name>dfs.namenode.http-address</name>
     <value>0.0.0.0:50070</value>
     <description>The address and port that the NameNode web UI will listen on.</description>
    </property>
</configuration>

編輯 master fille
sudo gedit /usr/local/hadoop/etc/hadoop/masters ，set master

新版 hadoop 預設是 worker 不是 slaves，編輯 worker file set => sudo gedit /usr/local/hadoop/etc/hadoop/workers
data1
data2
data3
重啟master

利用 virtiual box cli 背景執行 data1~3
cd  D: ，CD : \Oracle\VirtualBox (安裝位置)

.\VBoxManage startvm "data1" --type headless
.\VBoxManage startvm "data2" --type headless
.\VBoxManage startvm "data3" --type headless
檢查 => VBoxManage list runningvms
(要 shutdown 用 .\VBoxManage controlvm "您的虛擬機名稱" acpipowerbutton )
![背景執行 data1、2、3](./screen%20shot/Snipaste_2025-10-29_19-49-00.png)

透過 ssh data1、2、3測試，exit 可以離開
移除 hdfs 目錄 => sudo rm -rf /usr/local/hadoop/hadoop_data/hdfs
建立 datanode 儲存目錄 => mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode (master node 要建立 namenode)
更改目錄權限 => sudo chown -R hduser:hduser /usr/local/hadoop/

• master node
格式化 namenode hdfs => hadoop namenode -format
啟動 hadoop multi node cluster => start-all.sh
檢查各datanode 有啟動  => ssh data1、2、3 jps

## 第八章

下載 scala => wget <https://www.scala-lang.org/files/archive/scala-2.13.16.tgz>
解壓縮 => tar xvf scala-2.13.16.tgz
搬移 scala => sudo mv scala-2.13.16 /usr/local/scala
設定 scala 環境參數 => sudo gedit ~/.bashrc
export SCALA_HOME=/usr/local/scala
export PATH=$PATH:$SCALA_HOME/bin

reload bashrc => source ~/.bashrc
輸入 scala 進入，成功後 輸入 :q 離開

下載 spark => wget <https://dlcdn.apache.org/spark/spark-4.0.1/spark-4.0.1-bin-hadoop3.tgz>
解壓縮 => tar zxf spark-4.0.1-bin-hadoop3.tgz
搬移到 usr/local => sudo mv spark-4.0.1-bin-hadoop3 /usr/local/spark/
設定環境參數 => sudo gedit ~/.bashrc

# =========== Spark 4.0 ===========

export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin

啟動pyspark ( 離開使用 exit() )=> hduser@master:~/Downloads$ pyspark
移至 pyspark 設定檔案目錄 =>  cd /usr/local/spark/conf

複製 log4j2 樣板到 log4j2.prop => cp log4j2.properties.template log4j2.properties
編輯 log4j2.properties => sudo gedit log4j2.properties
把 rootLogger.level = info 改為 rootLogger.level = warn

建立 HDFS user 目錄 =>  hadoop fs -mkdir /user
建立 hduser 子目錄 => hadoop fs -mkdir /user/hduser
建立 test 子目錄 => hadoop fs -mkdir /user/hduser/test
建立 wordcount 子目錄 => hadoop fs -mkdir -p /usr/hduser/wordcount
建立 input 子目錄 => hadoop fs -mkdir -p /usr/hduser/wordcount/input

建立 HDFS FOLDER=> mkdir -p ~/wordcount/input
建立測試文黨 => cp /usr/local/hadoop/LICENSE.txt ~/wordcount/input
check => ll ~/wordcount/input
在 HDFS 建立目錄=> hadoop fs -mkdir -p /usr/hduser/wordcount/input
切換到 wordcount 測試 folder => cd ~/wordcount
上傳文字檔到 HDFS => hadoop fs -copyFromLocal LICENSE.txt  /user/hduser/wordcount/input
print => hadoop fs -ls /user/hduser/wordcount/input

本機執行pyspark => pyspark --master local[*]
查看目前執行模式 => sc.master
讀取本機檔案 => textFile = sc.textFile("file:/usr/local/spark/README.md")
顯示比數 => textFile.count()
讀取 HDFS file => textFile = sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE.txt")
顯示比數 => textFile.count()
離開pyspark => exit()

在Hadoop YARN 執行 pyspark => HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop pyspark --master yarn --deploy-mode client
檢查模式 => sc.master
在 Yarn hadoop 讀取 HDFS file => textFile = sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE.txt")
顯示比數 => textFile.count()
打開 ::8088 => 點 application 就可以看到 spark 正在 running

建置 spark standalone cluster env
從樣板中複製建立 spark-env.sh => cp /usr/local/spark/conf/spark-env.sh.template /usr/local/spark/conf/spark-env.sh
編輯 => sudo gedit /usr/local/spark/conf/spark-env.sh
export SPARK_MASTER_IP=master
export SPARK_WORKER_CORES=1
export SPARK_WORKER_MEMORY=512m
export SPARK_EXECUTOR_INSTANCES=4

將master 的 SPARK 程式複製到 data1
連線到data1 => ssh data1、2、3
建立spark 目錄 => sudo mkdir /usr/local/spark
變更擁有者為hduser => sudo chown hduser:hduser /usr/local/spark
離開 data1、2、3回到 master
使用scp 將 master 的 spark 程式複製到 data1 => sudo scp -r /usr/local/spark hduser@data1:/usr/local

在 spark standalone 執行 pyspark
編輯workers => gedit /usr/local/spark/conf/workers
data1
data2
data3
啟動 spark standalone cluster => /usr/local/spark/sbin/start-all.sh

在 spark standalone 執行 pyspark 程式 => pyspark --master spark://master:7077 --num-executors 1 --total-executor-cores 3 --executor-memory 512m
或 => pyspark --master spark://master:7077 --driver-memory 512m --conf spark.executor.memory=512m --conf spark.executor.cores=1

查看目前執行程式 => sc.master
讀取本機檔案 => textFile=sc.textFile("file:/usr/local/spark/README.md")
檢查 => textFile.count()

讀取hdfs 檔案 => textFile=sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE/txt")
read => textFile.count()

關閉spark => /usr/local/spark/sbin/stop-all.sh
關閉hadoop => stop-all.sh
關機

## 第九章

在master、data1、data2、data3 安裝anaconda => bash Anaconda......sh
加入anaconda 環境變數 => sudo gedit ~/.bashrc

# =========== Anaconda 3 ===========

export PATH=/home/hduser/anaconda3/bin:$PATH
export ANACONDA_PATH=/home/hduser/anaconda3
export PYSPARK_DRIVER_PYTHON=$ANACONDA_PATH/bin/jupyter
export PYSPARK_PYTHON=$ANACONDA_PATH/bin/python

# ==================================

重載環境變數 => source ~/.bashrc

建立 ipynotebook 工作目錄 => mkdir -p ~/pythonwork/ipynotebook
進到資料夾 => cd ~/pythonwork/ipynotebook/
在 IPython Notebook(Jupyter) 執行 pyspark => PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook" pyspark

![Jupyter](./screen%20shot/VirtualBox_master_29_10_2025_19_18_41.png)

輸入python指令
![Jupyter](./screen%20shot/VirtualBox_master_29_10_2025_20_01_21.png)

啟用 ipython notebook(jupyter) 在 hadoop yarn-client 模式 =>PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook" HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop pyspark --master yarn --deploy-mode client
![在 yarn-client 模式下開啟](./screen%20shot/Snipaste_2025-10-29_20-15-27.png)
![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-16-00.png)

在hadoop web 查看 pyspark app
![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-17-22.png)

使用 Stand Alone模式執行 => /usr/local/spark/sbin/start-all.sh

執行 jupyter notebook => PYSPARK_DRIVER_PYTHON=jupyter \
PYSPARK_DRIVER_PYTHON_OPTS="notebook" \
pyspark \
--master spark://master:7077 \
--num-executors 1 \
--total-executor-cores 2 \
--executor-memory 512m
![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-24-24.png)

執行後檢查 <http://master:8080/> =>
![Spark](./screen%20shot/Snipaste_2025-10-29_20-26-37.png)
