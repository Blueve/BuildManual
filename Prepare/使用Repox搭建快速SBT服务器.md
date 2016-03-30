# 使用Repox搭建快速SBT服务器
---

## 目的
由于国内连接SBT速度十分不稳定，因此本文将使用[Repox](https://github.com/Centaur/repox)来搭建一个自己的SBT服务器，试图解决SBT的连接问题。

## 前提
- 需要连接网络
- 需要使用Linux系统，本文中使用Ubuntu 14.04.1 LTS
- 各软件版本
    + SBT 0.13.11
    + Scala 2.10.6
    + JDK 1.8.0_77(由于Repox的原因，JDK7是无法使用的)

---
## 准备
### 更新 apt
`sudo apt-get update`

### 安装 Git
`sudo apt-get install git -y`

### 安装 Scala
`sudo apt-get install scala -y`

### 安装 SBT
由于Repox的原因，需要先使用SBT构建项目，因此需要先安装[SBT](http://www.scala-sbt.org/0.13/docs/Installing-sbt-on-Linux.html)

```Shell
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
sudo apt-get update
sudo apt-get install sbt
```
### 安装与配置JDK
将已下载好的jdk压缩包解压至/usr/lib/jvm下
`sudo tar -zxvf jdk-8u77-linux-x64.tar.gz -C /usr/lib/jvm`

配置环境变量
`sudo gedit ~/.bashrc`
在文件尾部添加
```Shell
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_79
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
sudo nano /etc/profile
```
执行`source ~/.bashrc`

### 安装 node.js
`sudo apt-get install nodejs -y`
链接nodejs至node
`ln -s /usr/bin/nodejs /usr/bin/node`

### 安装 npm
`sudo apt-get install npm -y`

### 安装 bower
`sudo npm install bower -g`

---
## Repox
### Repox 服务器
从源码编译打包
```
git clone https://github.com/Centaur/repox.git
cd repox/src/main/resources/admin
bower install
cd ../../../..
sbt assembly
```
sbt将会在target/scala-2.11/目录下生成repox-assembly-$VERSION.jar

运行
`java -Xmx512m -jar repox-assembly-$VERSION.jar`

由于上游仓库的连接情况会随着时间改变，因此，Repox开发者提供了[配置文件](http://repox.gtan.com:8078/admin/exportConfig)的更新，如果多次尝试SBT命令时仍不能解决依赖，可更新配置文件继续尝试。
只要将该配置文件放置于`/repox/src/main/resources/admin/`目录下，替换`repox.config.json`文件即可

### Repox 客户端
#### windows下使用SBT
向SBT的安装目录的Conf文件夹中的`sbtconfig.txt`添加
`-Dsbt.repository.config=xxx/sbt/conf/repo.properties`
(xxx代表SBT安装目录)

在Conf文件夹中添加`repo.properties`文件，文件内容如下：（xxx.xxx.xxx.xxx代表Repox服务器的ip地址）
```
[repositories]
local
repox-maven: http://xxx.xxx.xxx.xxx:8078/
repox-ivy: http://xxx.xxx.xxx.xxx:8078/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
```
#### ubuntu 14.04.1LTS下使用SBT
配置 ~/.sbt/repositories 文件，内容如下：（xxx.xxx.xxx.xxx代表Repox服务器的ip地址）
```
[repositories]
local
repox-maven: http://xxx.xxx.xxx.xxx:8078/
repox-ivy: http://xxx.xxx.xxx.xxx:8078/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
```

#### 其他
其他设置，请参考[此文](https://github.com/Centaur/repox/wiki/%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97)