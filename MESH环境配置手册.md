#MESH环境配置手册

##安装问题

主要参考：

http://mesh.cs.umn.edu/setup.html

其中Unix环境为Centos 7，因为不知道为什么在Ubuntu中搭建spark会各种报错？离谱

Java 8:java版本为oracle，而不是openjdk

```
user@host $ java -version
java version "1.8.0_92"
Java(TM) SE Runtime Environment (build 1.8.0_92-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.92-b14, mixed mode)
```

scala 2.10.5

maven 3.3.9

spark 1.6

若安装包找不到可以联系qq:974293785



安装步骤基本为

```
tar -zxvf ***.tar.gz

sudo vim /etc/profile
```

其中：

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_181
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
export MVN_HOME=/usr/java/maven/maven
export PATH=$PATH:$MVN_HOME/bin:$JAVA_HOME/jre/bin
export SCALA_HOME=****
export PATH=$PATH:$SCALA_HOME/bin
```



然后： source /etc/profile



接下来安装spark1.6,其中patch在[patch](https://raw.githubusercontent.com/mesh-umn/MESH/master/Graphx-Patch/meshGraphx_branch-1.6.patch)。其实在mesh中含有graphx1.6 jar包，因此也不需要去编译spark。

```
git clone https://github.com/apache/spark.git
cd spark
git checkout branch-1.6
git apply meshGraphx_branch-1.6.patch
```

build Graphx jar

```
./build/mvn -DskipTests clean package
```

将Graphx jar加入本地的maven repository

```
mvn install:install-file -Dfile=[path-to-file] -DgroupId=spark-graphx-mesh -DartifactId=spark-graphx-mesh -Dversion=1.6.0 -Dpackaging=jar

where: path-to-file: Path to the graphx-JAR to install
其中jar包在spark目录下的graphx目录中
```

然后切换至[MESH](https://github.com/mesh-umn/MESH/)目录下

```
mvn -DskipTests clean package
```



##代理问题

由于我所使用的服务器无法ping baidu.com,无法连接外网，因此我用本电脑当做代理，来实现下载连接，下面给出linux 代理方法。在此特别感谢hu‘ao同学所给予的帮助

1.修改http_proxy, https_proxy和ftp_proxy,参考[网站](https://www.cnblogs.com/EasonJim/p/9826681.html)

```bash
vim /etc/profile

export http_proxy=usrname:password@your_ip:端口

若无账号名称及密码，则
export http_proxy=your_ip:端口

https_proxy和ftp_proxy同理
```



2.在命令后面给出代理

```bash
-DproxySet=true -DproxyHost=your_ip -DproxyPort=端口

即：
mvn -DskipTests clean package -DproxySet=true -DproxyHost=your_ip -DproxyPort=端口
```



## 运行问题

由于MESH所依赖的库为spark1.6，因此在运行大图时，在内存方面的处理十分的挫，所以可能会出现GC overhead limit exceeded等问题,因此对运行命令进行了修改，如下

```
/home/***/MESH/spark/bin/spark-submit --conf spark.driver.maxResultSize=3G --driver-memory 32G --class umn.dcsg.examples.PageRankRunner examples/target/examples-1.0-SNAPSHOT-jar-with-dependencies.jar examples/data/com-orkut-MESH -1 "2D" 1 1 10 1 0.15
```

对于较小的内存会出现GC Error，对于较大的内存会由于clean时间太长而报错，本服务器使用32G则解决了此类问题。