# linux配置

### 免密登陆

两台机器：bigdata01 bigdata02

1. 首先在bigdata01上执行 ssh-keygen -t rsa
2. 执行这个命令以后，需要连续按 4 次回车键回到 linux 命令行才表示这个操作执行 结束，在按回车的时候不需要输入任何内容。
3. 执行以后会在\~/.ssh目录下生产对应的公钥和私钥文件
4. ![](<../../.gitbook/assets/image (24).png>)
5. 把公钥拷贝到需要免密码登录的机器上面
6. scp /.ssh/authorized\_keys bigdata02:\~/
7. `bigdata02: cat ~/a`uthorized\_keys `>> ~/.ssh/authorized_keys`
8. `ssh bigdata02`

### `配置别名`

vi /etc/hosts

```
192.168.182.100 bigdata01
192.168.182.101 bigdata02
192.168.182.102 bigdata03
```

### 时间同步

```
ntpdate -u ntp.sjtu.edu.cn
# yum install -y ntpdate

vi /etc/crontab
* * * * * root /usr/sbin/ntpdate -u ntp.sjtu.edu.cn
```

### PIP源配置

vim \~/.pip/pip.conf
