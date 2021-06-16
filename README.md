## 环境说明

| ip地址      | 主机名     | 操作系统版本 | RocketMQ版本 |  JDK版本  | maven版本 |
| ----------- | ---------- | :----------: | :----------: | :-------: | :-------: |
| 172.16.7.91 | rabbitmq01 |  centos 7.6  |    4.8.0     | 1.8.0_291 |    3.6    |

## 一、安装jdk

### 1.下载jdk

下载地址：https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html

![image-20210601170259948](https://i.loli.net/2021/06/16/xqC4DJrPWviEzhu.png)

### 2.安装jdk

新建目录/opt/java并将下载的包jdk-8u291-linux-x64.rpm 上传

```bash
[root@rabbitmq01 ~]# mkdir /opt/java 
[root@rabbitmq01 ~]# cd /opt/java
[root@rabbitmq01 java]# rpm -ivh jdk-8u291-linux-x64.rpm 
```

![image-20210601172403152](https://i.loli.net/2021/06/16/EWlSZ3UTbNPRmYt.png)

## 二、安装maven

### 1.下载安装包

```bash
[root@rabbitmq01 ~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

也可以到官网下载，下载地址：http://maven.apache.org/download.cgi

### 2.解压并配置环境变量

```bash
[root@rabbitmq01 ~]# tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local/
[root@rabbitmq01 ~]# mv /usr/local/apache-maven-3.6.3/ /usr/local/maven3.6
[root@rabbitmq01 ~]# sed -i '$a export PATH=$PATH:/usr/local/maven3.6/bin' /etc/profile && source /etc/profile
[root@rabbitmq01 ~]# mvn -v
```

![image-20210602094424067](https://i.loli.net/2021/06/16/SCgkX7KMIRhruow.png)

### 3.maven优化

#### 3.1配置阿里源

```bash
[root@rabbitmq01 ~]# cd /usr/local/maven3.6/conf/
[root@rabbitmq01 conf]# view settings.xml
```

修改后：

```bash
<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
```

![image-20210602100342700](https://i.loli.net/2021/06/16/DOJCty6P8cwoSFH.png)

#### 3.2指定资源下载路径

```bash
<localRepository>/usr/local/maven3.6/repository</localRepository>
```

![image-20210602100624371](https://i.loli.net/2021/06/16/LlGx79vEiPtqUpN.png)

#### 3.3指定JDK版本

将默认的1.4替换为实际的1.8

![image-20210602100754413](https://i.loli.net/2021/06/16/RT2tarNVyUOMpul.png)

![image-20210602100902226](https://i.loli.net/2021/06/16/KpsR5tdbQInF4iu.png)

## 三、RocketMQ安装

### 1.版本选取

下载地址：https://github.com/apache/rocketmq/tags

![image-20210601162652495](https://i.loli.net/2021/06/16/Bi7sZxG8dtWpNq2.png)

本次选择的版本为最新的4.8.0

### 2.下载安装包

```bash
[root@rabbitmq01 ~]# wget https://github.com/apache/rocketmq/archive/refs/tags/rocketmq-all-4.8.0.tar.gz
```

![image-20210601162809273](https://i.loli.net/2021/06/16/TQZkyWJ4DN8pPIK.png)

### 3.编译方式安装

```bash
[root@rabbitmq01 ~]# tar -zxvf rocketmq-all-4.8.0.tar.gz
[root@rabbitmq01 ~]# cd rocketmq-rocketmq-all-4.8.0/
[root@rabbitmq01 rocketmq-rocketmq-all-4.8.0]# mvn -Prelease-all -DskipTests clean install -U
```

### 4.免编译安装

除了使用maven编译安装方式外，还可以直接下载带bin的安装包，免编译安装，**步骤3和步骤4执行一个就行**。

下载地址：[**https://apache.claz.org/rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip**](https://apache.claz.org/rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip)

```bash
[root@rabbitmq01 ~]# mkdir rocketmq
[root@rabbitmq01 ~]# cd rocketmq/
[root@rabbitmq01 rocketmq]# wget https://apache.claz.org/rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip
```

![image-20210602152656561](https://i.loli.net/2021/06/16/DSwL6OJp8ay4HrK.png)

rocketmq-all-4.8.0-bin-release.zip带bin版本的介质无需maven编译，解压后即可使用。

### 5.调整runserver.sh参数

**调整前：**

```bash
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

**调整后：**

```bash
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

### 6.调整runbroker.sh参数

调整前：

```bash
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
```

调整后：

```bash
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

由于默认启动的最大内存比较大，需要修改小点保证服务能正常启动。

![image-20210609150427869](https://i.loli.net/2021/06/16/i2YwtJoWeTVSbN5.png)

### 7.启动namesrv

```bash
[root@rabbitmq01 bin]# nohup sh mqnamesrv &
```

![image-20210602163419421](https://i.loli.net/2021/06/16/t9PTXK7kp8FzmVN.png)

namesrv启动成功

### 8.启动broker

```bash
[root@rabbitmq01 bin]# nohup sh mqbroker -n localhost:9876 &
```

![image-20210602163942790](https://i.loli.net/2021/06/16/kyUQedpljEqzMWs.png)

broker节点启动成功

### 9.查看服务是否启动成功

```bash
[root@rabbitmq01 ~]# jps -l
41467 org.apache.rocketmq.broker.BrokerStartup
41214 org.apache.rocketmq.namesrv.NamesrvStartup
43887 sun.tools.jps.Jps
```

namesrv和broker启动正常

## 四、消息收发测试

### 1.生产者发送消息

```bash
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# export NAMESRV_ADDR=localhost:9876
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

![image-20210602173516852](https://i.loli.net/2021/06/16/SUbWjfgoculehtC.png)

消息测试，生产者发送大量的消息。

### 2.消费者消费消息

```bash
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

![image-20210602173741064](https://i.loli.net/2021/06/16/CRFifNKre2lScOv.png)

## 五、安装管理面板

### 1.介质下载

git下载介质

```bash
[root@rabbitmq01 ~]# yum -y install git
[root@rabbitmq01 ~]# git clone https://github.com/apache/rocketmq-externals.git

```

或者下载到本地再上传，下载地址：https://github.com/apache/rocketmq-externals/archive/refs/heads/master.zip

这里选择先下载再上传方式

### 2.解压

```bash
[root@rabbitmq01 ~]# unzip rocketmq-externals-master.zip 
```

![image-20210603100213749](https://i.loli.net/2021/06/16/KMA6sfNgj8t3u5z.png)

### 3.修改配置文件

```bash
[root@rabbitmq01 resources]# pwd
/root/rocketmq-externals-master/rocketmq-console/src/main/resources
[root@rabbitmq01 resources]# view application.properties
```

![image-20210602180337697](https://i.loli.net/2021/06/16/wXydf2pea76VgjU.png)

自定义应用端口和nameserver端口

### 4.编译

```bash
[root@rabbitmq01 rocketmq-console]# pwd
/root/rocketmq-externals-master/rocketmq-console
[root@rabbitmq01 rocketmq-console]# mvn clean package -Dmaven.test.skip=true
```

![image-20210603100550847](https://i.loli.net/2021/06/16/KpV4WX1COZb6fsR.png)

编译过程如图，中间过程省略

![image-20210603100434620](https://i.loli.net/2021/06/16/TFCeKJQ1VHhIcbD.png)

编译成功，console目录下会生成target目录

### 5.运行

```bash
[root@rabbitmq01 rocketmq-console]# cd target/
[root@rabbitmq01 target]# java -jar rocketmq-console-ng-2.0.0.jar
```

![image-20210603101055892](https://i.loli.net/2021/06/16/jfL7hlu4GtRrecq.png)

![image-20210603101227836](https://i.loli.net/2021/06/16/GMpEK7yQLxgDnwY.png)

### 6.访问console

![image-20210603102158602](https://i.loli.net/2021/06/16/oR2nM4qrCgcTWaQ.png)

![image-20210603102421953](https://i.loli.net/2021/06/16/NzBkMvlS6jeoV9t.png)

原因是跨域造成的，需要修改服务器中broker的配置，添加服务器IP即可

## 六、重启服务

### 1.关闭服务

```bash
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# pwd
/root/rocketmq/rocketmq-all-4.8.0-bin-release
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# sh bin/mqshutdown broker
No mqbroker running.
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# sh bin/mqshutdown namesrv
No mqnamesrv running.
```

![image-20210603103124687](https://i.loli.net/2021/06/16/Trz7FA4JEiO5kqm.png)

关闭服务时先停止broker再停止namesrv

### 2.修改broker配置

```bash
[root@rabbitmq01 ~]# sed -i '$a brokerIP1=172.16.7.91' /root/rocketmq/rocketmq-all-4.8.0-bin-release/conf/broker.conf 
```

### 3.开启服务

```bash
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# pwd
/root/rocketmq/rocketmq-all-4.8.0-bin-release
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# nohup sh bin/mqnamesrv &
[root@rabbitmq01 rocketmq-all-4.8.0-bin-release]# nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.conf &
```

启动服务时先启动nameserv再启动broker

![image-20210603103433451](https://i.loli.net/2021/06/16/6WRPhwMonKxJTfr.png)

启动完成

### 3.重新登陆console

![image-20210603103637384](https://i.loli.net/2021/06/16/G4bJ89uwpN3Qcfa.png)

## 七、自启动脚本

### 1.rocketmq服务脚本

```bash
[root@rabbitmq01 init.d]# pwd
/etc/rc.d/init.d
[root@rabbitmq01 init.d]# touch rocketmq
[root@rabbitmq01 init.d]# chmod u+x rocketmq
```

在/etc/rc.d/init.d下新建服务rocketmq，根据实际情况修改配置参数。

![image-20210607152658399](https://i.loli.net/2021/06/16/qrcVFMgfu9lCvk4.png)

### 2.rocketmq服务启停测试

```bash
[root@rabbitmq01 ~]# service rocketmq stop
[root@rabbitmq01 ~]# service rocketmq start
[root@rabbitmq01 ~]# service rocketmq restart
```

![image-20210607142150039](https://i.loli.net/2021/06/16/GEIhqf9t8vOdYNg.png)

### 3.rocketmq_console自启停脚本

```bash
[root@rabbitmq01 init.d]# touch rocketmq_console
[root@rabbitmq01 init.d]# chmod u+x rocketmq_console
[root@rabbitmq01 init.d]# more rocketmq_console 
```

![image-20210607152750668](https://i.loli.net/2021/06/16/7otVsNl5M4pnqmB.png)

### 4.rocketmq_console服务启停测试

```bash
[root@rabbitmq01 ~]# service rocketmq_console stop
[root@rabbitmq01 ~]# service rocketmq_console start
[root@rabbitmq01 ~]# service rocketmq_console restart
```

![image-20210607142725814](https://i.loli.net/2021/06/16/YUv2p8wOfkEljuI.png)

### 5.设置服务开机启动

```bash
[root@rabbitmq01 ~]# systemctl enable rocketmq
[root@rabbitmq01 ~]# systemctl enable rocketmq_console
[root@rabbitmq01 ~]# chkconfil --list
```

![image-20210607151229861](https://i.loli.net/2021/06/16/n1mUlwN5qLcEBsk.png)

设置完成后开机会自启动namesrv、broker和console服务


**搭建完成**
