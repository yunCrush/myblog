# RocketMQ环境搭建

### Linux**安装部署RocketMQ**

{% code overflow="wrap" %}
```powershell
cd /data/soft
#https://rocketmq.apache.org/download/
unzip rocketmq-all-4.7.1-bin-release.zip
cd bin
vi runserver.sh
# 定位到如下代码
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

# 修改　"-Xms -Xmx -Xmn"　参数
JAVA_OPT="${JAVA_OPT} -server -Xms512M -Xmx512M -Xmn256M -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
# 启动
nohup ./mqnamesrv &
# 查看日志
tail -f /root/logs/rocketmqlogs/namesrv.log
```
{% endcode %}

**2. 修改配置文件**

```
vi conf/broker.conf
# 使用如下配置文件
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH

storePathRootDir=/data/rocketmq/store
storePathCommitLog=/data/rocketmq/store/commitlog
namesrvAddr=127.0.0.1:9876
brokerIP1=192.168.202.128
brokerIP2=192.168.202.128
autoCreateTopicEnable=false

```

**3.修改 Broker JVM 参数**

```
cd bin
vi runbroker.sh 
#修改如下配置(配置前)
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
#配置后
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m"
```

**4.启动Broker**

```
cd bin
nohup ./mqbroker -c ../conf/broker.conf &
```

成功在 Linux 环境上安装了 RocketMQ NameServer 服务器与 Broker 服务器

**5.查看集群状态**

```
sh ./mqadmin clusterList -n 127.0.0.1:9876
```

### **Linux/win安装 RocketMQ-Console**

{% code overflow="wrap" %}
```
# linux
wget https://github.com/apache/rocketmq-externals/archive/rocketmq-console-1.0.0.tar.gz
tar -xf rocketmq-console-1.0.0.tar.gz
# 重命名，为了方便后续操作
# win 使用rocketmq-externals-rocketmq-console-1.0.0.zip， 解压

mv rocketmq-externals-rocketmq-console-1.0.0/rocketmq-console  rocketmq-console
```
{% endcode %}

**修改配置文件**

```
cd rocketmq-console
vi src/main/resources/application.properties
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>application.properties</p></figcaption></figure>

**使用 Maven 命令编译源代码**

`mvn clean package -DskipTests`&#x20;

编译后在 target 目录下会生成可运行的 jar 包,可以将生成的jar包放到/home/crush下。

**启动 RokcetMQ-Console**

`nohup java -jar rocketmq-console-ng-1.0.0.jar &`

在浏览器中输入 http://localhost:8080 查看是否安装成功

### Win-IDEA 中安装 RocketMQ

#### 1. NameSrv配置

1. **从 GitHub 上下载 RocketMQ 源码，并将其导入到 IEDA 中**
2.  ****

    <figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**Step2  namesrv/src/main/java/org/apache/rocketmq/namesrv/NamesrvStartup 设置环境变量 ROCKETMQ\_HOME**

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

**将 distribution/conf/logback\_namesrv.xml 文件拷贝到 Step2 中设置的主目录下**

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

**以 Debug 方法运行 NamesrvStartup,执行效果如下图所示,表示启动成功**

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

#### 2.Broker配置

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

对broker的环境变量进行配置

```
-c C:\Users\zhangyunfei\Desktop\RocketMQ\tmp\rocketmq\conf\broker.conf
ROCKETMQ_HOME=C:\Users\zhangyunfei\Desktop\RocketMQ\tmp\rocketmq
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**temp/rocketmq** 是rokcetmq的一个配置文件的位置。

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% code title="broker.conf" overflow="wrap" %}
```editorconfig
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH

storePathRootDir=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store
storePathCommitLog=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store\\commitlog
namesrvAddr=127.0.0.1:9876
brokerIP1=20.20.32.253
brokerIP2=20.20.32.253

autoCreateTopicEnable=true
# consume queue文件的存储路径
storePathConsumeQueue=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store\\consumequeue
# 消息索引文件的存储路径
storePathIndex=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store\\index
# checkpoint文件的存储路径
storeCheckpoint=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store\\checkpoint
# abort文件的存储路径
abortFile=C:\\Users\\zhangyunfei\\Desktop\\RocketMQ\\tmp\\rocketmq\\store\\abort
```
{% endcode %}

**全局替换配置文件中的变量**&#x20;

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
