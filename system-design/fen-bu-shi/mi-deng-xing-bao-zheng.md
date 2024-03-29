# 幂等性保证

1.  订单提交按钮如何防止重复提交？

    ```
    // 区分是用户主动提交还是重复提交，多次提交订单后端应当生成一个订单
    ```
2. 表单录入页面如何防止重复提交？
3. 微服务接口重试机制，如何防止对业务数据产生影响？
4. 支付时，发生网络抖动，由于网络问题的重试，支付只应当扣一次钱。
5. 什么情况下需要做幂等性的校验？重复提交，接口重试，前端抖动，并不是所有的接口都要保证幂等。
6. 返回前端token，后端校验这个token是否还存在，每处理一个请求，就将token失效，需要考虑处理失败的情况。

幂等性的核心思想是通过**唯一的业务单号**进行幂等性的保证

### Select

```
天然幂等
```

### Insert

```
有唯一业务单号的情况，如商品ID+用户ID，可以利用数据库唯一索引，再次插入的话，无法插入
也可使用分布式锁，保证接口的幂等性，业务执行完成后不进行锁的释放，让其自动过期释放
```

```
没有唯一业务单号的情况：如用户注册 点击多次
进入注册页面，后端生成统一token返回到前端隐藏域中，用户在提交时，将token一同传入后台，
使用token获取分布式锁，完成insert操作，对于同一个页面重复的请求，是无法获取到分布式锁的，
执行成功后并不释放锁，让分布式锁自动过期。
```

### Update

```
更新操作传入版本号，通过乐观锁实现幂等性
update set version=version+1, xxx=${xxx} where id=xxx and version = ${xxx}
```

### Delete

```
第一次已经删除，第二次删除也不会有影响
```

有唯一的业务单号就使用分布式锁，没有唯一的业务号进行操作，都使用token机制来保证幂等性
