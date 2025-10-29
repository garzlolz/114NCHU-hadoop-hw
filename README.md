# 　雲端計算作業

## 第三章

1. **安裝 guest addition**

   - 安裝完 Ubuntu 後，開啟 guest addition 會報錯「缺少 bzip2」。
   - 打開 Terminal，輸入：

     ```shell
     sudo apt update
     sudo apt install bzip2 tar
     ```

   - 安裝完成。
   - ![更新基本套件](./screen%20shot/VirtualBox_master_29_10_2025_19_51_04.png)

2. **設定雙向剪貼簿**

   - 關機，設定為雙向剪貼簿。
   - ![雙向剪貼簿](./screen%20shot/Snipaste_2025-10-29_19-52-50.png)

3. **檢查 Python 版本，安裝 Java**

   - 先檢查 Python 版本，再決定 Spark 與 Hadoop 版本。
   - 檢查指令：

     ```shell
     python3 -v  # 結果顯示 Python 3.12.3
     ```

---

## 第四章

1.  **安裝 Java**

    - 指令：

      ```shell
      java -V
      sudo apt install openjdk-17-jre-headless
      ```

    - ![檢查版本](./screen%20shot/VirtualBox_master_29_10_2025_19_54_59.png)

2.  **檢查 Java 安裝位置**

    - 目的是為了 bashrc 使用
    - 指令：

           ```shell
           update-alternatives --display java
           ```

      ![檢查java位置](/screen%20shot/image.png)

3.  **安裝 SSH 與 Rsync**

    - 安裝 SSH：

      ```shell
      sudo apt-get install ssh
      ```

    - 安裝 Rsync（重複打 SSH 是正確的，系統會提示已安裝）：

      ```shell
      sudo apt-get install ssh
      ```

4.  **設定 SSH Key**

    - Ubuntu 24.04 dsa 已棄用，改用 RSA 金鑰：

      ```shell
      ssh-keygen -t rsa -f ~/.ssh/id_rsa
      ll ~/.ssh
      cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
      ```

5.  **安裝 Hadoop (JDK 17 支援 3.3.6)**

    - 下載並安裝：

      ```shell
      wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
      sudo tar -zxvf hadoop-3.3.6.tar.gz
      sudo mv hadoop-3.3.6 /usr/local/hadoop
      ll /usr/local/hadoop
      ```

6.  **安裝 gedit**

    - 指令：

      ```shell
      sudo apt install gedit
      sudo apt install gedit-plugins
      ```

7.  **編輯 .bashrc**

    - 指令：

      ```shell
      sudo gedit ~/.bashrc
      ```

    - 增加以下內容至最底部：

      ```shell
      # =================================================================
      # HADOOOP & JAVA ENVIRONMENT VARIABLES (FIXED FOR JDK 17)
      # =================================================================
      export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
      export HADOOP_HOME=/usr/local/hadoop
      export PATH=$PATH:$JAVA_HOME/bin
      export PATH=$PATH:$HADOOP_HOME/bin
      export PATH=$PATH:$HADOOP_HOME/sbin
      export HADOOP_MAPRED_HOME=$HADOOP_HOME
      export HADOOP_COMMON_HOME=$HADOOP_HOME
      export HADOOP_HDFS_HOME=$HADOOP_HOME
      export YARN_HOME=$HADOOP_HOME
      export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
      export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib"
      export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_LIBRARY_PATH
      ```

      ![bashrc](./screen%20shot/Snipaste_2025-10-29_21-43-49.png)

    - 儲存並重新載入：

      ```shell
      source ~/.bashrc
      ```

8.  **修正 Hadoop 配置檔案**

    - hadoop-env.sh：

      ```shell
      sudo gedit /usr/local/hadoop/etc/hadoop/hadoop-env.sh
      # 搜尋 JAVA_HOME，修正為：
      export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
      ```

      ![修正 JAVA_HOME](./screen%20shot/Snipaste_2025-10-29_21-45-51.png)

    - core-site.xml：

      ```shell
      sudo gedit /usr/local/hadoop/etc/hadoop/core-site.xml
      ```

      ```xml
      <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
      </property>
      ```

    - yarn-site.xml：

      ```shell
      sudo gedit /usr/local/hadoop/etc/hadoop/yarn-site.xml
      ```

      ```xml
      <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
      </property>
      <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
      </property>
      ```

    - mapred-site.xml：

      ```shell
      gedit /usr/local/hadoop/etc/hadoop/mapred-site.xml
      ```

      ```xml
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
      ```

    - hdfs-site.xml：

      ```shell
      sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
      ```

      ```xml
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
      ```

9.  **建立與格式化 HDFS 目錄**

    - 建立儲存目錄：

      ```shell
      sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/namenode
      sudo mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode
      sudo chown hduser:hduser -R /usr/local/hadoop
      hadoop namenode -format
      ```

    - 放寬 Java 權限、解決 Java 9+ 模組化安全錯誤：

      ```shell
      export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.lang=ALL-UNNAMED"
      export HADOOP_OPTS="$HADOOP_OPTS --add-opens java.base/java.nio=ALL-UNNAMED"
      ```

10. **設定 HDFS Web UI port**

    - 預設為 <http://localhost:9870/>，改為 <http://localhost:50070>

      ```xml
      <property>
        <name>dfs.namenode.http-address</name>
        <value>0.0.0.0:50070</value>
        <description>The address and port that the NameNode web UI will listen on.</description>
      </property>
      ```

11. **啟動 Hadoop**

    ```shell
      start-all.sh
    ```

    ![啟動 hadoop](./screen%20shot/Snipaste_2025-10-29_21-53-19.png)

---

## 第五、六章

1. **虛擬機再製與網路設定**

- 關閉虛擬機後，複製 (Clone) 新機，並編輯網路介面卡 2，啟用並設為「內部網路」。
  ![內部網路](./screen%20shot/Snipaste_2025-10-29_22-00-45.png)
- 啟動 Ubuntu，編輯網路設定檔，採用 Netplan（新版 Ubuntu 已棄用 `/etc/network/interfaces`，且網卡已不再用 eth0/eth1 命名）。

- 編輯指令：

  ```shell
  sudo gedit /etc/netplan/01-network-manager-all.yaml
  ```

- 範例內容：

  ```yaml
  network:
    version: 2
    renderer: networkd
    ethernets:
      enp0s3: # 對外 NAT，用 DHCP
        dhcp4: true
        nameservers:
          addresses: [8.8.8.8, 8.8.4.4] # 強制 Google DNS
      enp0s8: # 內部集群 Host-Only
        dhcp4: no
        addresses: [192.168.56.101/24] # 靜態 IP
  ```

  ![更改靜態 ip](/screen%20shot/Snipaste_2025-10-29_21-55-58.png)

- 變更權限：

  ```shell
  sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
  ```

- 套用新設定與驗證：

  ```shell
  sudo netplan apply
  ip a
  ```

2. **設定 hostname 與 hosts 清單**

- 修改主機名稱：

  ```shell
  sudo hostnamectl set-hostname data1
  ```

- 編輯 hosts 檔案，新增節點：

  ```shell
  sudo gedit /etc/hosts
  ```

  ```shell
  192.168.56.100 master
  192.168.56.101 data1
  192.168.56.102 data2
  192.168.56.103 data3
  ```

  ![新增節點](/screen%20shot/Snipaste_2025-10-29_21-57-12.png)

3. **Hadoop 組態檔更新**

- core-site.xml：

  ```xml
  <property>
    <name>fs.default.name</name>
    <value>hdfs://master:9000</value>
  </property>
  ```

- yarn-site.xml：

  ```xml
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
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>

  ```

- mapred-site.xml：

  ```xml
  <property>
    <name>mapred.job.tracker</name>
    <value>master:54311</value>
  </property>
  ```

- hdfs-site.xml：

  ```xml
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
  ```

4. **多節點複製及啟動**

- 更新 net-tools 與複製設定至 data2、data3、master 節點，並更改 IP/hostname。
- 背景啟動虛擬機（windows cli）：

  ```shell
  VBoxManage startvm "data1" --type headless
  VBoxManage startvm "data2" --type headless
  VBoxManage startvm "data3" --type headless
  ```

  ![cli](./screen%20shot/Snipaste_2025-10-29_21-58-54.png)

- 檢查與關機：

  ```shell
  VBoxManage list runningvms
  VBoxManage controlvm "VM名" acpipowerbutton
  ```

- 測試 SSH、建立 datanode/namenode 目錄，格式化，並啟動 Hadoop multi-node cluster：

  ```shell
  hadoop namenode -format
  start-all.sh
  ssh data1 jps
  ssh data2 jps
  ssh data3 jps
  ```

5. **多節點 HDFS、YARN，主節點（master）與 worker 節點設置**

- master 節點 hdfs-site.xml 範例（additional）：

  ```xml
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
  ```

6. **設置 master/worker 檔案**

- masters：

  ```shell
  master
  ```

- workers（新版為 workers，不是 slaves）：

  ```shell
  data1
  data2
  data3
  ```

7. **利用 VirtualBox CLI 管理節點**

- Windows VirtualBox CLI 背景啟動資料節點。

8. **建立/移除及權限設定**

- 建立 datanode/namenode 目錄，並更改目錄權限：

  ```shell
  sudo rm -rf /usr/local/hadoop/hadoop_data/hdfs
  mkdir -p /usr/local/hadoop/hadoop_data/hdfs/datanode
  sudo chown -R hduser:hduser /usr/local/hadoop/
  ```

- 格式化 master node 的 namenode：

  ```shell
  hadoop namenode -format
  ```

9. **啟動多節點叢集並檢查**

- 啟動：

  ```shell
  start-all.sh
  ```

- 檢查每個 datanode 是否啟動（連線測試）：

  ```Shell
  ssh data1 jps
  ssh data2 jps
  ssh data3 jps
  ```

  ![jps](./screen%20shot/Snipaste_2025-10-29_22-03-16.png)

---

## 第八章

1. **安裝 Scala**

   - 下載 Scala：

     ```shell
     wget https://www.scala-lang.org/files/archive/scala-2.13.16.tgz
     ```

   - 解壓縮 Scala：

     ```shell
     tar xvf scala-2.13.16.tgz
     ```

   - 搬移到指定目錄：

     ```shell
     sudo mv scala-2.13.16 /usr/local/scala
     ```

   - 設定環境參數：

     ```shell
     sudo gedit ~/.bashrc
     export SCALA_HOME=/usr/local/scala
     export PATH=$PATH:$SCALA_HOME/bin
     ```

   - 重新載入環境，進入 Scala，測試後離開：

     ```shell
     source ~/.bashrc
     scala      # 若成功進入，輸入 :q 離開
     ```

2. **安裝 Spark**

   - 下載 Spark：

     ```shell
     wget https://dlcdn.apache.org/spark/spark-4.0.1/spark-4.0.1-bin-hadoop3.tgz
     ```

   - 解壓縮：

     ```shell
     tar zxf spark-4.0.1-bin-hadoop3.tgz
     ```

   - 搬移到目錄：

     ```shell
     sudo mv spark-4.0.1-bin-hadoop3 /usr/local/spark/
     ```

     ![spark](./screen%20shot/Snipaste_2025-10-29_22-04-20.png)

   - 設定環境參數：

     ```shell
     sudo gedit ~/.bashrc
     # =========== Spark 4.0 ===========
     export SPARK_HOME=/usr/local/spark
     export PATH=$PATH:$SPARK_HOME/bin
     ```

   - 重新載入：

     ```shell
     source ~/.bashrc
     ```

3. **Pyspark 啟動與設定 log**

   - 啟動 pyspark（離開用 `exit()`）：

     ```shell
     pyspark
     ```

   - 複製 log4j 樣板並編輯 log 等級：

     ```shell
     cd /usr/local/spark/conf
     cp log4j2.properties.template log4j2.properties
     sudo gedit log4j2.properties
     # 將 rootLogger.level = info 改為 rootLogger.level = warn
     ```

     ![log4j](./screen%20shot/Snipaste_2025-10-29_22-05-36.png)

4. **建立 HDFS 測試目錄與檔案並上傳**

   - 建立 HDFS user 目錄：

     ```shell
     hadoop fs -mkdir /user
     hadoop fs -mkdir /user/hduser
     hadoop fs -mkdir /user/hduser/test
     hadoop fs -mkdir -p /usr/hduser/wordcount/input
     ```

   - 建立本機資料夾與檔案：

     ```shell
     mkdir -p ~/wordcount/input
     cp /usr/local/hadoop/LICENSE.txt ~/wordcount/input
     ll ~/wordcount/input
     ```

   - 上傳至 HDFS：

     ```shell
     hadoop fs -copyFromLocal LICENSE.txt /user/hduser/wordcount/input
     hadoop fs -ls /user/hduser/wordcount/input
     ```

5. **執行 Pyspark 測試與讀取資料**

   - 本機模式啟動：

     ```shell
     pyspark --master local[*]
     ```

   - 查看模式與讀取本機檔案：

     ```shell
     sc.master
     textFile = sc.textFile("file:/usr/local/spark/README.md")
     textFile.count()
     ```

   - 讀取 HDFS 檔案：

     ```shell
     textFile = sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE.txt")
     textFile.count()
     ```

   - 離開 pyspark：

     ```shell
     exit()
     ```

6. **在 Hadoop YARN 執行 Pyspark**

   - 啟動：

     ```shell
     HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop pyspark --master yarn --deploy-mode client
     sc.master
     textFile = sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE.txt")
     textFile.count()
     ```

     ![spark](./screen%20shot/Snipaste_2025-10-29_22-11-46.png)

   - Spark 運算可從 8088 查看 application 運作。

7. **建置 Spark Standalone Cluster**

   - 複製 spark-env.sh 樣板並編輯參數：

     ```shell
     cp /usr/local/spark/conf/spark-env.sh.template /usr/local/spark/conf/spark-env.sh
     sudo gedit /usr/local/spark/conf/spark-env.sh
     export SPARK_MASTER_IP=master
     export SPARK_WORKER_CORES=1
     export SPARK_WORKER_MEMORY=512m
     export SPARK_EXECUTOR_INSTANCES=4
     ```

   - 建立 workers 檔案：

     ```shell
     gedit /usr/local/spark/conf/workers
     # 內容：
     data1
     data2
     data3
     ```

   - 複製 spark 到 data1、2、3，建立目錄設定權限，使用 scp 複製：

     ```shell
     sudo mkdir /usr/local/spark
     sudo chown hduser:hduser /usr/local/spark
     sudo scp -r /usr/local/spark hduser@data1:/usr/local
     ```

   - 啟動 Spark standalone cluster：

     ```shell
     /usr/local/spark/sbin/start-all.sh
     ```

8. **Spark Standalone 執行 Pyspark 程式測試**

   - 啟動：

     ```shell
     pyspark --master spark://master:7077 --num-executors 1 --total-executor-cores 3 --executor-memory 512m
     # 或
     pyspark --master spark://master:7077 --driver-memory 512m --conf spark.executor.memory=512m --conf spark.executor.cores=1
     ```

   - 檢查模式與讀取檔案：

     ```shell
     sc.master
     textFile = sc.textFile("file:/usr/local/spark/README.md")
     textFile.count()
     textFile = sc.textFile("hdfs://master:9000/user/hduser/wordcount/input/LICENSE.txt")
     textFile.count()
     ```

   - 關閉 Spark：

     ```shell
     /usr/local/spark/sbin/stop-all.sh
     ```

   - 關閉 Hadoop：

     ```shell
     stop-all.sh
     ```

   - 關機。

---

## 第九章

1. **在 master、data1、data2、data3 安裝 Anaconda**

   - 執行安裝腳本：

     ```shell
     bash Anaconda.XX.XX.sh
     ```

   - 設定 Anaconda 環境變數，編輯 `.bashrc`：

     ```shell
     sudo gedit ~/.bashrc
     ```

   - 加入以下內容：

     ```shell
     # =========== Anaconda 3 ===========
     export PATH=/home/hduser/anaconda3/bin:$PATH
     export ANACONDA_PATH=/home/hduser/anaconda3
     export PYSPARK_DRIVER_PYTHON=$ANACONDA_PATH/bin/jupyter
     export PYSPARK_PYTHON=$ANACONDA_PATH/bin/python
     # ==================================
     ```

   - 重新載入環境變數：

     ```shell
     source ~/.bashrc
     ```

2. **建立 IPython Notebook 工作目錄並啟動 Jupyter**

   - 建立工作目錄：

     ```shell
     mkdir -p ~/pythonwork/ipynotebook
     cd ~/pythonwork/ipynotebook/
     ```

   - 執行 Pyspark 並開啟 Jupyter Notebook：

     ```shell
     PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook" pyspark
     ```

     ![Jupyter](./screen%20shot/VirtualBox_master_29_10_2025_19_18_41.png)

   - 介面示意與 Python 指令可在 Jupyter 中運行。
     ![Jupyter](./screen%20shot/VirtualBox_master_29_10_2025_20_01_21.png)

3. **在 Hadoop Yarn-client 模式下啟用 Jupyter**

   - 啟動命令：

     ```shell
     PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS="notebook" HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop pyspark --master yarn --deploy-mode client
     ```

     ![在 yarn-client 模式下開啟](./screen%20shot/Snipaste_2025-10-29_20-15-27.png)
     ![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-16-00.png)

   - 可在 Hadoop Web UI 監控 Pyspark 的應用程式運行。
     ![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-17-22.png)

4. **Spark Standalone 模式執行 Jupyter Notebook**

   - 啟動 Spark Standalone cluster：

     ```shell
     /usr/local/spark/sbin/start-all.sh
     ```

   - 執行 Jupyter Notebook 並連接 Spark Standalone：

     ```shell
     PYSPARK_DRIVER_PYTHON=jupyter \
     PYSPARK_DRIVER_PYTHON_OPTS="notebook" \
     pyspark \
     --master spark://master:7077 \
     --num-executors 1 \
     --total-executor-cores 2 \
     --executor-memory 512m
     ```

     ![Jupyter](./screen%20shot/Snipaste_2025-10-29_20-24-24.png)

   - 可透過 <http://master:8080/> 監控 Spark Web UI。
     ![Spark](./screen%20shot/Snipaste_2025-10-29_20-26-37.png)
