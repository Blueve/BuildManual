# Windows环境编译Hadoop.0.20.205

## 准备

- 源代码 [[Hadoop-0.20.205.0](https://github.com/apache/hadoop/releases/tag/release-0.20.205.0)]
- Apache Ant [[Ant](http://ant.apache.org/)]
- JDK >= 1.6
- Cygwin [[Cygwin](http://www.cygwin.com/)]

## 配置过程

1. 配置JAVA环境变量

  - 添加`JAVA_HOME`
  - 添加JRE的bin目录到Path

2. 配置Ant环境变量
  
  - 添加`ANT_HOME`
  - 添加Ant的bin目录到Path

3. 配置Cygwin

  - 安装时需选择ssl与ssh(最好完全安装)
  - 添加Cygwin的bin目录到Path

## 编译前准备

1. 打开Hadoop源码目录

2. 打开`build.xml`

  - 找到并注释
  ```xml
  <exec executable="sed" inputstring="${os.name}" outputproperty="nonspace.os">
  	<arg value="s/ /_/g"/>
  </exec>
  ```

  - 找到并注释
  ```xml
  <exec executable="sh">
  	<arg line="src/saveVersion.sh ${version} ${build.dir}"/>
  </exec>
  <exec executable="sh">
  	<arg line="src/fixFontsPath.sh ${src.docs.cn}"/>
  </exec>
  ```

3. 打开src/contrib/gridmix/src/java/org/apache/hadoop/mapred/gridmix/Gridmix.java

  - 找到方法
  ```java
  private <T> String getEnumValues(Enum<? extends T>[] e)
  ```
  将所有的`Enum<? extends T>`改为`Enum<?>`

## 编译

1. 在Hadoop源代码根目录打开命令提示符(CMD)

2. 输入并执行`ant`

3. 等待编译完成

4. 编译后的文件存放在`build/hadoop-core-0.20.205.1.jar`