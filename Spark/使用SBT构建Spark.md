# 使用SBT构建Spark

## 准备工作

- 安装SBT以及配置相关环境

可参考[使用Repox搭建快速SBT服务器](https://github.com/Blueve/BuildManual/blob/master/Prepare/%E4%BD%BF%E7%94%A8Repox%E6%90%AD%E5%BB%BA%E5%BF%AB%E9%80%9FSBT%E6%9C%8D%E5%8A%A1%E5%99%A8.md)

## Windows

### 命令行

- 编译并打包
```bash
sbt -Pyarn -Phadoop-2.6 -DskipTests assembly
```

- 编译后的Jar为
```
\assembly\target\scala-2.*\spark-assembly-*-SNAPSHOT-hadoop*.jar
```

### IntelliJ IDEA

- 使用Import

- 选择Spark源码目录

- 选择SBT方式构建

- 等待

- Build Make Project

## Ubuntu 14.04 LTS

- 待补充