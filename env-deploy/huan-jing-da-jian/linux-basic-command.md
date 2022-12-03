---
description: 常用命令，后序有待补充。
---

# Linux常用命令

### 1.linux常识性指令

```
# 显示帮助
uname --help
# 输出内核版本信息
uname -v                    
# 显示版本信息并退出
uname --version                    
# 等于who -m 登陆用户的信息
who am i                    
# 查询当前登录在系统上的用户信息
who
# root登陆具有所有权限
"#"
# 普通用户没什么权限
$
# -p 递归创建目录
mkdir -p /a/b/c/d
# 修改权限命令
chmod +x /home/crush/text.txt
# chown 更改属主，属组,crush即用户名
chown crush temp        
chown .crushgroup temp
```

### 2.文件的传输

**windows本地安装git bash使用**

_1.从远处(服务器)复制文件到本地目录_

```
$ scp root@192.168.45.129:/usr/local/tmp/test.md ./
```

_2.从远处(服务器)复制目录到本地目录_

```
$ scp -r root@192.168.45.129:/usr/local/tmp ./
```

_3.上传本地文件到服务器指定目录_

```
$ scp -r /opt/test.tar root@192.168.45.129:/usr/local
```

_4.上传本地目录到服务器指定目录_

```
$ scp -r /opt/soft/text root@192.168.45.12:/usr/local
```

_5.windows文件传输到服务器_

```
yum install lrzsz
直接将文件拖入到命令行窗口即可
```

### 3.查看端口与进程

```
# 查看es端口9200是否存在
netstat -lnp | grep 9200

# 查看所有socket连接数量
netstat -a

# 查看es进程是否存在
ps -ef | grep elasticsearch
```

```
# xargs指令从标准数据流中构造并执行一行行的指令,-I是查找替换，echo查看指令是否有误
ls | xargs -I GG echo "mv GG prefix_GG"
ls | xargs -I GG mv GG prefix_GG

# | 匿名管道  mkfifo指令可以创建一个命名管道
```

```
#远程登录的 ssh 指令；

#查看网络接口的 ifconfig 指令；

#测试网络延迟的 ping 指令；
ping IP/domain

#可以交互式调试和服务端的 telnet 指令；
telnet Ip port

#两个 DNS 查询指令 host 和 dig
host -t AAAA www.yuncrush.com
dig www.yuncrush.com
```

## 4.git代理

```
# 解除ssl验证
git config --global http.sslVerify "false"

# 添加代理
git config --global http.proxy http://127.0.0.1:1080
 
git config --global https.proxy http://127.0.0.1:1080
#取消全局代理：
git config --global --unset http.proxy
 
git config --global --unset https.proxy
```

## 5.zip分割与合并

```
# 将tempdir目录下的文件打包成多个小包 -P password带密码,小包的前缀名字myfile.xx
zip -P password -r -s 1024m myfile.zip tempdir/
#将获得的多个压缩包合并成一个
zip -F myfiles.zip --out single-archive.zip
#得到single-archive.zip包，对其进行解压即可。
unzip -P password singl-archive.zip -d ./
```

### 6.启动java项目

```
nohup java -jar test.jar >/dev/null 2>&1 &
nohup 后台运行， & 不锁定窗口，但是关闭窗口，程序结束
>/dev/null 2>&1 &
标准输出： > 代码为1 >>累加不覆盖
标准错误输出： 2> 代码为2 2>>累加不覆盖
>/dev/null 2>&1 标准输出与错误输出都输出到/dev/null &1即等价于代码1的输出位置
1表示标准输出，2标准错误输出。&1表示引用标准输出的那个文件。
command > /dev/null  等价于command 1 > /dev/null 
```
