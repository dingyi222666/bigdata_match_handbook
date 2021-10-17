## hadoop动态添加删除节点

### 修改主机名

```sh
hostnamectl set-hostname slave3 && bash
```

### 设置时区为上海时区

```sh
tzselect
# 5 > 9 > 1 > 1
# TZ='Asia/Shanghai'; export TZ
# 输入到/etc/profile
# 或者 echo "TZ='Asia/Shanghai'; export TZ" >> /etc/profile && source /etc/profile)
source /etc/profile
# 让更改生效
```

### 关闭防火墙

```sh
systemctl stop firewalld 
```

### 修改hosts信息

```sh
vim /etc/hosts
# hosts name
# 在全部主机上操作
```



### 设置每隔十分钟同步时间

```shell
crontab -e
*/10 * * * * /usr/ntpdate master
```



### 设置master到slave3的免密登录

```shell
# master操作
ssh-copy-id slave3
# 输入密码同意即可
```



### 安装java和hadoop

```sh
mkdir -p /usr/java && tar -zxvf jdk-8u221-linux-x64.tar.gz -C /usr/java && mkdir -p /usr/hadoop && tar hadoop-2.7.7.tar.gz -zxvf -C /usr/hadoop

#
echo -e "#JAVA_HOME\nexport JAVA_HOME=/usr/java/jdk1.8.0_221\nexport PATH=\$PATH:\$JAVA_HOME/bin\n#HADOOP_HOME\nexport HADOOP_HOME=/usr/hadoop/hadoop-2.7.7\nexport PATH=\$PATH:\$HADOOP_HOME/bin\nexport PATH=\$PATH:\$HADOOP_HOME/sbin" >> /etc/profile && source /etc/profile
```



### 新节点动态上线

#### 启动集群

```shell
cd /usr/hadoop/hadoop-2.7.7/ && start-dfs.sh && start-yarn.sh
```

#### 配置主节点的hdfs-site.xml

```shell
cd /usr/hadoop/hadoop-2.7.7/etc/hadoop && vim hdfs-site.xml
```

然后添加如下内容

```xml
  <!-- 允许加入集群的节点列表 -->
  <property>
    <name>dfs.hosts</name>
    <value>/usr/hadoop/hadoop-2.7.7/etc/hadoop/datanode-allow.list</value>
  </property>
  <!-- 拒绝加入集群的节点列表 -->
  <property>
    <name>dfs.hosts.exclude</name>
    <value>/usr/hadoop/hadoop-2.7.7/etc/hadoop/datanode-deny.list</value>
  </property>
```

#### 修改datanode-allow.list文件

```shell
cd /usr/hadoop/hadoop-2.7.7/etc/hadoop && echo -e "master\nslave1\nslave2\nslave3" > datanode-allow.list
```

#### 修改节点配置文件

```sh
cd /usr/hadoop/hadoop-2.7.7/etc/hadoop && echo -e "\nslave3" >> slaves 
```



#### 启动slave3 DataNode和NodeManager进程

```shell
cd /usr/hadoop/hadoop-2.7.7 && hadoop-daemon.sh start datanode && yarn-daemon.sh start nodemanager
```



#### master刷新查看节点状态

````shell
cd /usr/hadoop/hadoop-2.7.7 && hdfs dfsadmin -refreshNodes && yarn rmadmin -refreshNodes
````

