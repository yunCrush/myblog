---
description: 'mysq-canal同步的环境搭建 mysql:8.0 canal: 1.1.6'
---

# MySQL-Canal-Redis

## 1. MySQL准备工作

```
// 辅助表，整合springboot使用 注意这里使用的是t_user2
CREATE TABLE `t_user2` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `userName` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4
// 当前主机的二进制日志
show master status;
mysql配置文件位置：/etc/my.cnf
// 添加
server_id = 1
binlog-format = ROW
```

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption><p>my.cnf</p></figcaption></figure>

* ROW模式 除了记录sql语句之外，还会记录每个字段的变化情况，能够清楚的记录每行数据的变化历史，但会占用较多的空间。
* STATEMENT模式只记录了sql语句，但是没有记录上下文信息，在进行数据恢复的时候可能会导致数据的丢失情况；
* MIX模式比较灵活的记录，理论上说当遇到了表结构变更的时候，就会记录为statement模式。当遇到了数据更新或者删除情况下就会变为row模式；

**重启MySQL: systemctl restart mysqld**

1. 开启binlog写入功能   &#x20;

```
// 查看binlog写入功能是否开启
show variables like "log_bin"
```

2\. 授权canal连接MySQL账号

```
DROP USER 'canal'@'%';
CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';  
grant all privileges on *.* to 'canal'@'%';
FLUSH PRIVILEGES;
SELECT * FROM mysql.user;
```

## 2.安装canal

```
wget https://github.com/alibaba/canal/releases/download/canal-1.1.6/canal.deployer-1.1.6.tar.gz
```

```
mkdir -p /data/soft/mycanal       
tar -zxvf canal.deployer-1.1.6.tar.gz 
//这里以官网example下的instance.properties举例，正常修改canal.properties文件，修改前应当备份
cd example
vi instance.properties
// 修改mysql主机地址
 canal.instance.master.address=127.0.0.1:3306
// 修改mysql中添加的canal用户
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
启动canal即可 /bin/start
```

启动服务：./startup.sh

验证启动成功：生成canal.pid或者/data/soft/mycanal/logs/canal/canal.log&#x20;

<figure><img src="../../.gitbook/assets/image (6) (2).png" alt=""><figcaption><p>canal-position</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (2).png" alt=""><figcaption><p>canal-conf</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (4) (1).png" alt=""><figcaption><p>instance.properties</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>启动成功</p></figcaption></figure>

## 踩坑

启动报错，自MySQL 8.0.3开始，身份验证插件默认使用caching\_sha2\_password

<figure><img src="../../.gitbook/assets/image (5) (2).png" alt=""><figcaption><p>canal启动报错</p></figcaption></figure>

解决方案：

```
select host,user,plugin from mysql.user;
alter user 'canal'@'%' IDENTIFIED with mysql_native_password BY 'canal';  
```

## 3.整合SpringBoot

**项目启动报错1：mysql连接数量过多**

```
MySql host blocked because of many connection errors;
unblock with 'mysqladmin flush-hosts'
```

<pre><code><strong>// 解决方法:
</strong>whereis mysqladmin
/usr/bin/mysqladmin flush-hosts -h192.168.202.128 -P3306 -uroot -p123456
</code></pre>

**项目启动报错2：MySQL 8.0 Public Key Retrieval is not allowed 错误的解决方法**

由于加密方式修改配置文件末尾添加 \&allowPublicKeyRetrieval=true

```

spring.datasource.url=jdbc:mysql://192.168.202.128:3306/db2021?useUnicode=true&characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true
```
