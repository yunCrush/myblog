---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: false
  outline:
    visible: false
  pagination:
    visible: true
---

# MySQL环境篇

```
//关闭selinux: vi /etc/selinux/config
//SELINUX=disabled

//安装第三方依赖包
yum install perl -y
yum install net-tools -y

// 移除mariadb
rpm -qa | grep maridb
rpm -e ...

//安装mysql相关rpm包,yum缓存位置
cd /var/cache/yum/x86_64/7/mysql80-community/packages/
rpm -ivh mysql-XXX-common
rpm -ivh mysql-XXX-libs
rpm -ivh mysql-XXX-client
rpm -ivh mysql-XXX-server
//////////////////////////////////
// yum安装会自动安装相关的依赖
yum install mysql-community-server.x86_6
///////////////////////////////////

//修改 /var/lib/mysql目录权限
chmod -R 777 /var/lib/mysql
//初始化
mysqld --initialize
chmod -R 777 /var/lib/mysql/*

// 启动数据库
systemctl start mysqld
//查看临时密码
grep "temporary password" /var/log/mysqld.log
//登陆修改密码
mysql -u root -p 
alter user user() identified by "123456";
//允许使用远程登陆
use mysql;
update user set host='%' where user='root';
flush privileges;
//修改配置文件，添加
vim /etc/my.cnf
character_set_server = utf8
bind-address = 0.0.0.0
//重启服务，开放3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

问题：MySQL GPG秘钥过期

```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
//再安装
yum install mysql-community-server.x86_6
```
