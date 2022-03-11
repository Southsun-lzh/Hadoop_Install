# Hadoop安装文档

本教程由wzg撰写，用于安装Hadoop，所需工具与软件版本预览：

* jdk 1.8.0_321
* Hadoop 2.10.1
* Finalshell 3.9.3.4
* VMware Workstation 16.1.2 build-17966106

## 本机

* 下载Finalshell 

  [官方下载地址](http://www.hostbuf.com/downloads/finalshell_install.exe)

* VMware Workstation (许可证请自行解决)

  [官方下载地址](https://www.vmware.com/go/getworkstation-win)

## 主控机

#### 下载CentOS 7镜像

​	[下载地址](http://isoredirect.centos.org/centos/7/isos/x86_64)

​	安装教程暂时搁置

​	登录主控机后：

* net-tools 安装

  ```
  yum install -y net-tools
  ```

  一路按y即可安装

  ```
  ifconfig    #查看你的虚拟机ip地址，后面从物理机ssh连接它
  ```

  然后回到物理机，打开 Finalshell (以管理员身份打开），以ssh连接到虚拟机。

  ```
  mkdir /usr/lib/java  #在/usr/lib/路径下创建java文件夹
  ```

  

* 从物理机下载 jdk8 然后上传 jdk8 到/usr/lib/java文件夹

  [下载地址](https://download.oracle.com/otn/java/jdk/8u321-b07/df5ad55fdd604472a86a45a217032c7d/jdk-8u321-linux-x64.tar.gz)  需要登录才能下载

* 解压jdk

```shell
tar -xvf jdk-8u321-linux-x64.tar.gz
```



* 添加环境变量

```shell
vi /etc/profile

### vi 命令简单使用指南   ###
# vi -h 查看使用帮助
# 最直接的用法就是 vi 后面路径
# 例如：vi /etc/profile
# 确定后就会进入预览模式，你可以用方向键移动光标，但是不能进行编辑。若想编辑，按一次 i 键，就会   进入编辑模式
# 编辑结束，按ESC 键 跳到命令模式，然后输入退出命令：
# :w 保存文件但不退出vi 编辑
# :w! 强制保存，不退出vi 编辑
# :w file 将修改另存到 file 中，不退出vi 编辑 
# :wq 保存文件并退出vi 编辑
# :wq! 强制保存文件并退出vi 编辑
# :q! 不保存文件并强制退出vi 编辑
# :e! 放弃所有修改，从上次保存文件开始在编辑 

# 最常用的就是 :wq 保存并退出
```

shift+g 直接定位到文档末尾，将以下内容添加:

```txt
export JAVA_HOME=/usr/lib/java/jdk1.8.0_321
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

* 刷新状态：

  ```
  source /etc/profile
  ```

* 验证是否成功配置环境变量

  ```
  java -version
  ```

  成功样例：

  ```txt
  java version "1.8.0_321"
  Java(TM) SE Runtime Environment (build 1.8.0_321-b07)
  Java HotSpot(TM) 64-Bit Server VM (build 25.321-b07, mixed mode)
  ```

* 修改主机名（可选）

  ```
  hostnamectl set-hostname wzg #‘wzg’为你想取的名字，然后重新连接即可刷新
  ```

* 修改Host文件

  ```
  vi /etc/hosts
  ```

  ```
  192.168.13.129 wzg   #格式： ip 主机名
  192.168.13.130 slave1
  ```

  追加以上内容(ip和名称修改成自己的)

#### 下载Hadoop

去Hadoop的官网下载文件： 

```
https://hadoop.apache.org/releases.html 
```

选择2.10.1版本

先在根目录下创建一个文件夹:

```
mkdir /hadoop
cd /hadoop   
```



上传到服务机/hadoop目录解压：

```
tar -xvf hadoop-2.10.1.tar.gz
```

切换到以下目录：

```
cd hadoop-2.10.1/etc/hadoop
```

编辑以下文件（可选，为了保险）：

```
vi hadoop-env.sh  #添加Java环境路径
export JAVA_HOME=/usr/lib/java/jdk1.8.0_321    #这条是替换，找一下
vi yarn-env.sh
export JAVA_HOME=/usr/lib/java/jdk1.8.0_321    #这条是添加的，在你喜欢的位置的添加
```

回到hadoop-2.10.1目录

```shell
cd ..
cd ..
pwd       #查看当前工作目录是否是/hadoop/hadoop-2.10.1
mkdir hdfs
mkdir hdfs/name
mkdir hdfs/data    #创建这些文件夹用来保存文件
```

找到 /etc/hadoop/core-site.xml 文件，编辑以下内容到，如下格式（框里面是要添加的内容<configuration>不用加）：

<configuration>  #注意wzg:9000 的'wzg'改成你自己的主机名

```xml
	<property>
        <name>fs.defaultFS</name>
        <value>hdfs://wzg:9000</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
```

</configuration>

同理 /etc/hadoop/hdfs-site.xml 文件

<configuration>

```xml
	<property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/hadoop/hadoop-2.10.1/hdfs/name</value>
	</property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/hadoop/hadoop-2.10.1/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
```

</configuration>

同理 /etc/hadoop/yarn-site.xml 文件

<configuration>

```xml
	<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>wzg</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>2048</value>
	</property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
	</property>
```

</configuration>

同理 /etc/hadoop/mapred-site.xml 文件(需要重命名)

<configuration>

```
	 <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
     </property>
```

</configuration>

编辑 /etc/hadoop/slaves 文件，添加你的slave节点机ip 地址。如下:

```
192.168.13.130
10.70.122.105
```

修改主控机的host文件，在 /etc/hosts路径下

```shell
vi /etc/hosts

#格式如下：ip 主机名
# 192.168.13.132 wzg
# 192.168.13.130 slave1
# 10.70.122.105 slave2
```

让自己免密登录

```shell
ssh-keygen -t dsa -f ~/.ssh/id_dsa     #生成公钥
cp /root/.ssh/id_dsa.pub /root/.ssh/authorized_keys  #让自身免密登录
```

让主控机连接节点机免密登录

```
将id_dsa.pub里面的内容追加到节点机的/root/.ssh/authorized_keys文件中
若没有这个文件，可以在节点机也输入以下命令：
ssh-keygen -t dsa -f ~/.ssh/id_dsa     #生成公钥
cp /root/.ssh/id_dsa.pub /root/.ssh/authorized_keys  #让自身免密登录
再试一次即可
```



* 启动Hadoop:

  ```shell
  bin/hdfs namenode -format   #初始化Hadoop
  sbin/./start-all.sh			#启动服务
  ```

* 关闭防火墙（为了能正常访问，后续可以自行配置防火墙）

  ```
  systemctl stop firewalld
  ```

  

* 在物理浏览器访问目标主机的8088端口验证是否打开了网页

  ```
  ip:port
  ```

  

#### 一些解决部分问题的命令

```
export HADOOP_SSH_OPTS="-p 1234"      #临时将Hadoop ssh 连接slave端口设置为1234
```

在 /root/.ssh/下新建一个文件名为 config 的文件,添加以下内容（请自行更改参数）：

```
#slave2  备注
Host slave2  #你的主机名
Hostname 10.70.122.105   #主机地址,一般是ip地址
Port 7777    #ssh连接端口
User root    #ssh登录时的用户名
IdentityFile ~/.ssh/id_dsa   #私钥存放地址，可删除这一行
```

这样你的每次ssh连接可直接使用如下命令

```
ssh host   #Host为你的主机名
```

