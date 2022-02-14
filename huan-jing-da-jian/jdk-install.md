---
description: 最快的速度在linux上面安装jdk8。
---

# JDK安装

## 1.拉取资源包

```
# 选择jdk8的安装位置文件
cd  /usr/local/

# 拉取资源包
cd /usr/local/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"

# 解压资源包
tar zxvf  jdk-8u141-linux-x64.tar.gz

# 重命名文件
mv jdk-8u141-linux-x64  java8
```

## 2.修改配置文件

```
# 修改配置文件
vim /etc/profile

# 插入以下配置
export JAVA_HOME=/usr/local/java8
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 解析配置文件
source /etc/profile

# 验证javac
```

