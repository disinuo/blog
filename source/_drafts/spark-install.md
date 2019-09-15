
## spark安装
```shell
# 1.安装scala
 sudo apt-get install scala

#装好后在命令行输入scala有如下输出即安装成功

Welcome to Scala version 2.XX.XX (OpenJDK 64-Bit Server VM, Java 1.8.X).
Type in expressions to have them evaluated.
Type :help for more information.
scala>
# :q可以退出此模式

# 2.下载spark
 wget http://mirrors.shu.edu.cn/apache/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz
# 3.解压
sudo mkdir /usr/local/spark
sudo tar -zxvf spark-2.4.0-bin-hadoop2.7.tgz -C /usr/local/
sudo mv /usr/local/spark-2.4.0-bin-hadoop2.7 /usr/local/spark
# 4.路径配置
#找一下scala被安装在哪
whereis scala
#修改配置文件
sudo vim ~/.bashrc #/etc/profile也可以，/etc/profile是全局的

#然后在文件最后添加如下代码
export SPARK_HOME=/usr/local/spark/
export SCALA_HOME=/usr/share/scala-2.11
export PATH=$PATH:${SPARK_HOME}/bin:${SCALA_HOME}/bin

#然后立刻更新文件
source ~/.bashrc
```

配好啦~

测试一下

```shell
#1.
spark-shell
#输出大概长这样
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.0
      /_/
Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_181)
Type in expressions to have them evaluated.
Type :help for more information.
scala>

'`
#2.用自带函数测试，算一下pi

cd /usr/local/spark/bin/

./run-example SparkPi 5

#输出很多行，主要有

Successfully created connection to /10.8.4.7:46328 after 28 ms (0 ms spent in bootstraps)
TaskSetManager:54 - Starting task 4.0 in stage 0.0

Pi is roughly 3.1497662995325992

```
这样就是安装成功啦~
## spark配置
```shell
#1.进入spark目录
cd /usr/local/spark/conf
#2.从配置模板copy创建配置文件
cp spark-env.sh.template spark-env.sh
#3.在spark-env.sh末尾添加下面配置
# export SCALA_HOME=/home/spark/workspace/scala-2.10.4

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export SCALA_HOME=/usr/share/scala-2.11/
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
SPARK_MASTER_IP=master
SPARK_MASTER_PORT=7077
SPARK_MASTER_HOST=master
SPARK_LOCAL_IP=master
SPARK_LOCAL_DIRS=/usr/local/spark
SPARK_DRIVER_MEMORY=1G

#保存的时候会提示此文件是readonly，退出的时候用`:wq!`就可以啦~

#4.配置slave
vi /etc/hosts
#然后添加ip和名称
#比如
xx.xx.xx.xx slave1
xx.xx.xx.xx master

#5.把配好的文件发给slave们~
#因为我们的配置文件实在/usr/local下的，master没有权限写入，
#因此先发送到临时路径，再在slave服务器下将其转移到目标路径
#==在master上操作
sudo scp -r /usr/local/spark/conf/spark-env.sh ubuntu@slave1:~/temp_conf.sh
#==在slave1上操作
sudo mv temp_conf.sh /usr/local/spark/conf/spark-env.sh

```
---
================================
## 配置spark用python的编程环境
