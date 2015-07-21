#Hadoop 2.7.1 完全分布式搭建

## 搭建Hadoop所需软件

- vmware ubuntu 14.04
- Oracle JDK8
- OPENSSH-SERVER
- BIND9
- NFS
- Hadoop2.7.1[下载](http://mirrors.cnnic.cn/apache/hadoop/common/)  [文档](http://hadoop.apache.org/docs/r2.7.1/)


## 安装过程

### 节点设定
| role        | name          | IP|
|------------|-----------|--------|
| NameNode| master.grid|192.168.64.133|
| DataNode| slave1.grid|192.168.64.134|
| DataNode | slave2.grid|192.168.64.135|

### 在vmware中安装ubuntu14.04

### 在ubuntu中安装jdk     [参考](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get)

- 安装OracleJDK8

  ```Shell
  sudo add-apt-repository ppa:webupd8team/java
  sudo apt-get update
  sudo apt-get install oracle-java8-installer
  sudo apt-get install orecle-java8-set-default
  ```

- 添加环境变量`sudo nano /etc/environment`

- 添加新一行JAVA_HOME="/usr/lib/jvm/java-8-oracle/jre/bin/java"

- 重新加载配置`source /etc/environment`

- 查看环境变量`echo $JAVA_HOME`

### 安装OPENSSH-SERVER

`sudo apt-get install openssh-server`

### 为每个节点设置静态IP

`sudo nano /etc/network/interfaces`

添加以下内容
```
auto eth0
iface eth0 inet static
address 192.168.64.133
gateway 192.168.64.2
netmask 255.255.255.0
network 192.168.64.0
broadcast 192.168.19.2
```

### BIND9安装与配置

Bind是一款开放源码的DNS服务器软件，用于映射节点的IP与域名，只需在DNS服务器上安装即可

- 安装`sudo apt-get install bind9`

- 编辑配置文件`sudo nano /etc/bind/named.conf.local`

  添加以下配置
  ```
  zone "grid" IN{
          type master;
          file "/var/cache/bind/db.grid";
          allow-update {none; };
  };
  zone "64.168.192.in-addr.arpa" IN{
          type master;
          file "/var/cache/bind/db.64.168.192";
          allow-update {none; };
  };
  ```

- 在/var/cache/bind目录下添加db.grid

  ```
  ;
  ; BIND reverse data file for broadcast zone
  ;
  $TTL    604800
  @       IN      SOA     localhost. root.localhost. (
                          42              ; Serial
                          3H              ; Refresh
                          15H             ; Retry
                          1W              ; Expire
                          1D )            ; Negative Cache TTL
  ;
  @       IN      NS      localhost.
  master  IN      A       192.168.64.133
  slave1  IN      A       192.168.64.134
  slave2  IN      A       192.168.64.135
  ```

- 在/var/cache/bind目录下添加db.64.168.192

  ```
  ;
  ; BIND reverse data file for broadcast zone
  ;
  $TTL    604800
  @       IN      SOA     localhost. root.localhost. (
                          42              ; Serial
                          3H              ; Refresh
                          15H             ; Retry
                          1W              ; Expire
                          1D )            ; Negative Cache TTL
  ;
  @       IN      NS      localhost.
  133     IN      PTR     master.grid.
  134     IN      PTR     slave1.grid.
  135     IN      PTR     slave2.grid.
  ```

- 重启NAS服务器

  `sudo /etc/init.d/bind9 restart`

- 在所有节点的/etc/resolvconf/resolv.conf.d/base里添加`nameserver 192.168.64.133`

- 在所有节点中执行`sudo resolvconf -u`

- 测试，输入`host slave1.grid`

  显示`slave1.grid has address 192.168.64.134`即为配置成功


### NFS的安装与使用

文件服务器，用于在各个节点之间共享文件

- 服务器配置

  1. 安装NFSSERVER`sudo apt-get install nfs-kernel-server`

  1. 编辑配置文件`sudo nano /etc/exports`，添加需要共享的文件路径

    在exports文件中，加入`/home/hadoop/.ssh*(rw,async,no_root_squash)`
    > 常用参数
    *：允许访问的网段，可以替换成例如192.168.64.0/5
    rw：可读写的权限；
    ro：只读的权限；
    no_root_squash：登入到NFS主机的用户如果是ROOT用户，他就拥有ROOT的权限root_squash：在登入NFS主机使用目录的使用者如果是root时，那么这个使用者的权限将被压缩成为匿  名  使用者，通常他的UID与GID都会变成nobody那个身份
    all_squash：不管登陆NFS主机的用户是什么都会被重新设定为nobody。
    anonuid：将登入NFS主机的用户都设定成指定的userid,此ID必须存在于/etc/passwd中。
    anongid：同anonuid，但是变成groupID就是了！
    sync：资料同步写入存储器中。
    async：资料会先暂时存放在内存中，不会直接写入硬盘。
    insecure：允许从这台机器过来的非授权访问。

  1. 重启NFS服务`sudo service nfs-kernel-server restart`

- 客户端配置

  1. 在客户机上，安装 nfs-common`sudo apt-get -y install nfs-common`
  1. 在客户机上，挂载nfs目录`sudo mount -t nfs master.grid:/home/hadoop/ /mnt`
  1. 在客户机上，添加开机自动挂载NFS目录`sudo nano /etc/fstab`
    加入`master.grid:/home/hadoop/ /mnt nfs defaults 0 0`

### 使用NFS做ssh免密码连接配置 [参考](http://blog.csdn.net/lichangzai/article/details/8646227)

- 在各节点操作生成一个RSA密钥对

  `ssh-keygen -t rsa`

- 整合authorized_keys密钥

  NFS服务器所在节点`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
  其他节点`cat ~/.ssh/id_rsa.pub >> /mnt/.ssh/authorized_keys`
  查看整合后的文件`cat authorized_keys`

- 在各节点创建共享目录文件authorized_keys的软连接

  `ln -s /mnt/.ssh/authorized_keys ~/.ssh/authorized_keys`

- 测试ssh免密码连接

  `ssh 192.168.64.135 date`
  结果如下`Thu Jul 16 22:01:49 PDT 2015`

### 安装Hadoop

- 修改主机名`sudo nano/etc/hostname`

- 安装hadoop至文件夹中，本次解压至~/下，`tar-xvzf~/hadoop-2.7.1.tar.gz`

- 配置hadoop

  1. hadoop-2.7.1/etc/hadoop/hadoop-env.sh

    修改`export JAVA_HOME=/usr/lib/jvm/java-8-oracle`

  1. hadoop-2.7.1/etc/hadoop/core-site.xml

    ```XML
    <configuration>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/home/hadoop/hadoop-2.7.1/tmp</value>
        </property>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://master.grid:9000</value>
        </property>
    </configuration>
    ```
    在hadoop-2.7.1/etc/hadoop/中添加tmp文件夹
  
  1. hadoop-2.7.1/etc/hadoop/hdfs-site.xml

    ```XML
    <configuration>
        <property>
            <name>dfs.name.dir</name>
            <value>/home/hadoop/hadoop-2.7.1/data</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
    </configuration>
    ```
    在hadoop-2.7.1/etc/hadoop/中添加data文件夹

  1. hadoop-2.7.1/etc/hadoop/mapred-site.xml
  
    ```XML
    <configuration>
        <property>
            <name>mapred.job.tracker</name>
            <value>master.grid:9001</value>
        </property>
    </configuration>
    ```

  1. hadoop-2.7.1/etc/hadoop/slaves

    ```
    slave1.grid
    slave2.grid
    ```

- 向各节点复制hadoop

  ```Shell
  cat ~/hadoop-2.7.1/etc/hadoop/slaves| awk '{print "scp -rp hadoop-2.7.1 hadoop@"$1":/home/hadoop"}' > scp.sh
  chmod u+x scp.sh
  cat scp.sh
  ./scp.sh
  ```

- 启动hadoop

  1. 格式化namenode

    ```Shell
    hadoop@master:~$ cd /home/hadoop/hadoop-2.2.0/bin/ 
    hadoop@master:~/hadoop-2.7.1/bin$ ./hdfs namenode -format 
    ```

  1. 启动hdfs

    ```Shell
    hadoop@master:~/hadoop-2.7.1/bin$ cd ../sbin/
    hadoop@master:~/hadoop-2.7.1/sbin$ ./start-dfs.sh 
    ```

  1. 设置环境变量

    `sudo nano /etc/profile`
    ```Shell
    export HADOOP_HOME=/home/hadoop/hadoop-2.7.1
    export PATH=$HADOOP_HOME/bin:$PATH
    ```