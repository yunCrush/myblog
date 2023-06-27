# 配置163yum源

```
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
//备份替换体统repo
cp CentOS7-Base-163.repo /etc/yum.repos.d/ 
cd /etc/yum.repos.d/ 
mv CentOS-Base.repo CentOS-Base.repo.bak 
mv CentOS7-Base-163.repo CentOS-Base.repo
//更新缓存
yum clean all
yum makecache
yum update
```

yum 相关命令

```
yum list | grep mysql
```
