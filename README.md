### flume 安装及配置

##### 1. 安装jdk并配置

```
#如果服务器上只有一个jdk版本的话可以用这种配置

vi /etc/profile
#添加
export JAVA_HOME=/dbtool/java_software/java_jdk/jdk1.8.0_171
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=$JAVA_HOME/jre

#保存退出
source /etc/profile


#如果服务器上有多个jdk版本, 不建议这样操作, 可以用下面创建flume env的方式进行
```

如果是AIX系统, 参考另外一遍文章[AIX 上安装 JDK](https://github.com/yabolu/AIX-JDK/blob/master/README.md)

##### 2. 安装flume

```
tar zxvf apache-flume-1.8.0-bin.tar.gz
mv apache-flume-1.8.0-bin flume
cd flume

#创建flume jdk虚拟环境, 跟网上方式略有不同, 不需要创建FLUME_HOME环境变量

cd conf
cp flume-env.sh.template flume-env.sh

vi flume-env.sh

#将JAVA_HOME注释去掉, 并改为:
export JAVA_HOME=/dbtool/java_software/java_jdk/jdk1.8.0_171
#如果是AIX系统, 我的配置是:
export JAVA_HOME=/usr/java7_64
#保存退出即可

#因为我的项目需求需要对采集上来的每条日志添加物理IP和实例名, 所以我添加了flume自定义拦截器

cd lib/ 
拷贝 flume-interceptor.jar 到这个目录

```

##### 3. 配置及启动

* 采集flume配置(需要部署到各个被采集方)

    ```
    vi vi multi-exec-multi-file-multi-avro.conf(目前未使用)
        
    ###添加内容
    f1.sources = r1 r2
    f1.channels = c1 c2
    f1.sinks = k1 k2
    
    #define sources
    f1.sources.r1.type = exec
    f1.sources.r1.command =tail -f /dbtool/python_software/data.log
    
    f1.sources.r2.type = exec
    f1.sources.r2.command =tail -f /dbtool/python_software/data2.log
    
    #define channels
    f1.channels.c1.type = file
    f1.channels.c1.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint
    f1.channels.c1.dataDirs = /dbtool/python_software/flume/file-channel/data
    
    f1.channels.c2.type = file
    f1.channels.c2.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint2
    f1.channels.c2.dataDirs = /dbtool/python_software/flume/file-channel/data2
    
    #define sink
    f1.sinks.k1.type = avro
    f1.sinks.k1.hostname = 192.168.31.51
    f1.sinks.k1.port =44444
    
    f1.sinks.k2.type = avro
    f1.sinks.k2.hostname = 192.168.31.51
    f1.sinks.k2.port = 55555
    
    #bind sources and sink to channel
    f1.sources.r1.channels = c1
    f1.sinks.k1.channel = c1
    
    f1.sources.r2.channels = c2
    f1.sinks.k2.channel = c2
    
    
    vi vi multi-exec-multi-file-multi-kafka.conf(目前使用这种)
        
    ###添加内容
    f1.sources = r1 r2 r3 r4 r11
    f1.channels = c1 c2 c3 c4
    f1.sinks = k1 k2 k3 k4
    
    #define sources
    f1.sources.r1.type = exec
    f1.sources.r1.command =tail -f /dbtool/python_software/data1.log
    #r1的拦截器
    f1.sources.r1.interceptors = i1
    f1.sources.r1.interceptors.i1.type = com.zhb.flume.AppendIPBuilder
    f1.sources.r1.interceptors.i1.ipAddress = 192.168.31.61
    f1.sources.r1.interceptors.i1.instanceName = rac1
    
    f1.sources.r11.type = exec
    f1.sources.r11.command =tail -f /dbtool/python_software/data11.log
    #r11的拦截器
    f1.sources.r11.interceptors = i11
    f1.sources.r11.interceptors.i11.type = com.zhb.flume.AppendIPBuilder
    f1.sources.r11.interceptors.i11.ipAddress = 192.168.31.61
    f1.sources.r11.interceptors.i11.instanceName = rac11
    
    f1.sources.r2.type = exec
    f1.sources.r2.command =tail -f /dbtool/python_software/data2.log
    #r2的拦截器
    f1.sources.r2.interceptors = i2
    f1.sources.r2.interceptors.i2.type = com.zhb.flume.AppendIPBuilder
    f1.sources.r2.interceptors.i2.ipAddress = 192.168.31.61
    f1.sources.r2.interceptors.i2.instanceName = +ASM1
    
    f1.sources.r3.type = exec
    f1.sources.r3.command =tail -f /dbtool/python_software/data3.log
    #r3的拦截器
    f1.sources.r3.interceptors = i3
    f1.sources.r3.interceptors.i3.type = com.zhb.flume.AppendIPBuilder
    f1.sources.r3.interceptors.i3.ipAddress = 192.168.31.61
    f1.sources.r3.interceptors.i3.instanceName =
    
    f1.sources.r4.type = exec
    f1.sources.r4.command =tail -f /dbtool/python_software/data4.log
    #r4的拦截器
    f1.sources.r4.interceptors = i4
    f1.sources.r4.interceptors.i4.type = com.zhb.flume.AppendIPBuilder
    f1.sources.r4.interceptors.i4.ipAddress = 192.168.31.61
    f1.sources.r4.interceptors.i4.instanceName =
    
    #define channels
    f1.channels.c1.type = file
    f1.channels.c1.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint
    f1.channels.c1.dataDirs = /dbtool/python_software/flume/file-channel/data
    
    f1.channels.c2.type = file
    f1.channels.c2.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint2
    f1.channels.c2.dataDirs = /dbtool/python_software/flume/file-channel/data2
    
    f1.channels.c3.type = file
    f1.channels.c3.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint3
    f1.channels.c3.dataDirs = /dbtool/python_software/flume/file-channel/data3
    
    f1.channels.c4.type = file
    f1.channels.c4.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint4
    f1.channels.c4.dataDirs = /dbtool/python_software/flume/file-channel/data4
    
    #define sink
    f1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
    f1.sinks.k1.kafka.topic = tp-db-log
    f1.sinks.k1.kafka.bootstrap.servers = 192.168.31.51:9092
    f1.sinks.k1.kafka.flumeBatchSize = 20
    f1.sinks.k1.kafka.producer.acks = 1
    f1.sinks.k1.kafka.producer.linger.ms = 1
    f1.sinks.k1.kafka.producer.compression.type = snappy
    
    f1.sinks.k2.type = org.apache.flume.sink.kafka.KafkaSink
    f1.sinks.k2.kafka.topic = tp-asm-log
    f1.sinks.k2.kafka.bootstrap.servers = 192.168.31.51:9092
    f1.sinks.k2.kafka.flumeBatchSize = 20
    f1.sinks.k2.kafka.producer.acks = 1
    f1.sinks.k2.kafka.producer.linger.ms = 1
    f1.sinks.k2.kafka.producer.compression.type = snappy
    
    f1.sinks.k3.type = org.apache.flume.sink.kafka.KafkaSink
    f1.sinks.k3.kafka.topic = tp-grid-log
    f1.sinks.k3.kafka.bootstrap.servers = 192.168.31.51:9092
    f1.sinks.k3.kafka.flumeBatchSize = 20
    f1.sinks.k3.kafka.producer.acks = 1
    f1.sinks.k3.kafka.producer.linger.ms = 1
    f1.sinks.k3.kafka.producer.compression.type = snappy
    
    f1.sinks.k4.type = org.apache.flume.sink.kafka.KafkaSink
    f1.sinks.k4.kafka.topic = tp-os-log
    f1.sinks.k4.kafka.bootstrap.servers = 192.168.31.51:9092
    f1.sinks.k4.kafka.flumeBatchSize = 20
    f1.sinks.k4.kafka.producer.acks = 1
    f1.sinks.k4.kafka.producer.linger.ms = 1
    f1.sinks.k4.kafka.producer.compression.type = snappy
    
    #bind sources and sink to channel
    f1.sources.r1.channels = c1
    f1.sources.r11.channels = c1
    f1.sinks.k1.channel = c1
    
    f1.sources.r2.channels = c2
    f1.sinks.k2.channel = c2
    
    f1.sources.r3.channels = c3
    f1.sinks.k3.channel = c3
    
    f1.sources.r4.channels = c4
    f1.sinks.k4.channel = c4        
    ```

    创建所需的文件夹
    
    ```
    mkdir -p /dbtool/python_software/flume/file-channel/checkpoint
    mkdir -p /dbtool/python_software/flume/file-channel/data
    
    mkdir -p /dbtool/python_software/flume/file-channel/checkpoint2
    mkdir -p /dbtool/python_software/flume/file-channel/data2
    
    mkdir -p /dbtool/python_software/flume/file-channel/checkpoint3
    mkdir -p /dbtool/python_software/flume/file-channel/data3
    
    mkdir -p /dbtool/python_software/flume/file-channel/checkpoint4
    mkdir -p /dbtool/python_software/flume/file-channel/data4

    ```


    启动采集flume
    
    ```
    //前台启动
    bin/flume-ng agent \
    --conf conf \
    --name f1 \
    --conf-file conf/multi-exec-multi-file-multi-kafka.conf \
    -Dflume.root.logger=DEBUG,console
    
    //后台启动
    bin/flume-ng agent \
    --conf conf \
    --name f1 \
    --conf-file conf/multi-exec-multi-file-multi-kafka.conf \
    1>/dev/null 2>&1 &
    
    //停止
    ps -ef | grep flume
    kill -9 pid
    ```
    
* 配置收集flume(一般部署到采集方)(目前未使用)

    ```
    vi vi multi-avro-multi-file-multi-kafka.conf
    
    ###添加内容
    f2.sources = r2 r22
    f2.channels = c2 c22
    f2.sinks = k2 k22
    
    #define sources
    f2.sources.r2.type = avro
    f2.sources.r2.bind = 192.168.31.51
    f2.sources.r2.port = 44444
    f2.sources.r2.channels = c2
    
    
    f2.sources.r22.type = avro
    f2.sources.r22.bind = 192.168.31.51
    f2.sources.r22.port = 55555
    f2.sources.r22.channels = c22
    
    #define channels
    f2.channels.c2.type = file
    f2.channels.c2.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint
    f2.channels.c2.dataDirs = /dbtool/python_software/flume/file-channel/data
    
    f2.channels.c22.type = file
    f2.channels.c22.checkpointDir = /dbtool/python_software/flume/file-channel/checkpoint2
    f2.channels.c22.dataDirs = /dbtool/python_software/flume/file-channel/data2
    
    #define sinks
    f2.sinks.k2.channel = c2
    f2.sinks.k2.type = org.apache.flume.sink.kafka.KafkaSink
    f2.sinks.k2.kafka.topic = mytopic
    f2.sinks.k2.kafka.bootstrap.servers = localhost:9092
    f2.sinks.k2.kafka.flumeBatchSize = 20
    f2.sinks.k2.kafka.producer.acks = 1
    f2.sinks.k2.kafka.producer.linger.ms = 1
    f2.sinks.k2.kafka.producer.compression.type = snappy
    
    f2.sinks.k22.channel = c22
    f2.sinks.k22.type = org.apache.flume.sink.kafka.KafkaSink
    f2.sinks.k22.kafka.topic = mytopic2
    f2.sinks.k22.kafka.bootstrap.servers = localhost:9092
    f2.sinks.k22.kafka.flumeBatchSize = 20
    f2.sinks.k22.kafka.producer.acks = 1
    f2.sinks.k22.kafka.producer.linger.ms = 1
    f2.sinks.k22.kafka.producer.compression.type = snappy
    ```
    
    启动收集flume
    
    ```
    //前台启动
    bin/flume-ng agent \
    --conf conf \
    --name f2 \
    --conf-file conf/avro-memory-logger_copy.conf \
    -Dflume.root.logger=DEBUG,console
    
    //后台启动
    bin/flume-ng agent \
    --conf conf \
    --name f2 \
    --conf-file conf/multi-avro-multi-file-multi-kafka.conf \
    1>/dev/null 2>&1 &
    
    //停止
    ps -ef | grep flume
    kill -9 pid
    ```






