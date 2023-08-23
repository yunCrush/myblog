# ES写入优化

<figure><img src="../../.gitbook/assets/es-write-principle.jpg" alt=""><figcaption></figcaption></figure>

1. 将刷新时间间隔设置为-1
2. 分片设置为1个，副本设置为0个
3. translog设置为异步，落盘时间，并配置合适的内存大小

```
  settingMap.put("index.refresh_interval", "-1");
  settingMap.put("index.translog.flush_threshold_size", "512mb");
  settingMap.put("index.translog.durability", "async");
  settingMap.put("index.translog.sync_interval", "60s");
```

```
// 写入数据后需要恢复
    settingMap.put("index.refresh_interval", "1s");
```
