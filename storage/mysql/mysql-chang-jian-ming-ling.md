# MySQL常见命令

## 1. 设置相关

```
# 登陆: mysql -u root -p密码
```

```
create table T(ID int primary key, c int);
 update T set c=c+1 where ID=2;
```

```
mysql执行顺序：
from table (join on)-> where ->group by (avg,sum) ->
->having ->order by -> select -> limit -> final
```
