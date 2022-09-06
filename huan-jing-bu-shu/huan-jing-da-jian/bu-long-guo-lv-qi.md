# 布隆过滤器

## 1.Rebloom(编译安装)

```
 
# 下载 编译 安装Rebloom插件
wget https://github.com/RedisLabsModules/rebloom/archive/v2.2.2.tar.gz
# 解压 
tar -zxvf v2.2.2.tar.gz
cd RedisBloom-2.2.2
# 若是第一次使用 需要安装gcc++环境
make
# redis服启动添加对应参数 这样写还是挺麻烦的
# rebloom_module="/usr/local/rebloom/rebloom.so"
# daemon --user ${REDIS_USER-redis} "$exec $REDIS_CONFIG --loadmodule # $rebloom_module --daemonize yes --pidfile $pidfile"
# 记录当前位置
pwd
# 进入reids目录 配置在redis.conf中 更加方便
vim redis.conf
# :/loadmodule redisbloom.so是刚才具体的pwd位置 cv一下
loadmodule /xxx/redis/redis-5.0.8/RedisBloom-2.2.2/redisbloom.so
# 保存退出
wq
# 重新启动redis-server 我是在redis中 操作的 若不在请写出具体位置 不然会报错
redis-server redis.conf
# 连接容器中的 redis 服务 若是无密码 redis-cli即可
redis-cli -a 密码
# 进入可以使用BF.ADD命令算成功
```

## 2.Rebloom(docker版)

```
docker run -p 6379:6379 --name=redis6379bloom -d redislabs/rebloom
```

```
docker exec -it redis6379bloom /bin/bash
```

```
redis-cli
```

![rebloom operation](<../../.gitbook/assets/image (38).png>)

```
bf.reserve key error_rate的值 initial_size 的值 
默认的error_rate是 0.01，
默认的initial_size是 100。
bf.add key 值
bf.exists key 值
bf.madd 一次添加多个元素    
bf.mexists 一次查询多个元素是否存在
```
