# Java

[TOC]

## JDK环境配置

### CentOS6 JDK配置
#### 检查Linux自带的OPENJDK
1. 查看已经安装的JAVA版本信息
    
    ```shell
    java -version
    ```
    
2. 查看JDK的信息
    
    ```shell
    rpm -qa | grep java
    ```
    
3. 卸载已经安装的JAVA
    
    ```shell
    yum remove java-1.6.0.xxxxxxx.xxx.xx
    ```
    
4. 卸载另外一个
    
    ```shell
    yum remove tzdata-java-xxxxx.x.xxx.xx
    ```
    
5. 安装并配置JDK
    
    ```shell
    vi /etc/profile
    # 在文件最尾添加
    JAVA_HOME=/usr/local/java/jdk1.8.0_131
    JRE_HOME=$JAVA_HOME/jre
    PATH=$PATH:$JAVA_HOME/bin
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export JAVA_HOME
    export JRE_HOME
export PATH
    export CLASSPATH
    
    source /etc/profile
    ```
    
    

