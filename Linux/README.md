---
coverY: 0
---

# Linux删除文件磁盘空间未释放

&#x20;      在Linux环境中，很容易出现一种情况，文件删除了但是磁盘空间未释放，句柄泄漏，导致磁盘空间打满，因此文档主要看下如何定位该问题以及针对本次case如何修复



问题定位及修复：

1.在机器上执行 &#x20;

```sh
lsof -n | grep deleted
```

2.kill 进程

```sh
kill -9 ${pid}
```

3.批量kill进程

```shell
lsof -n | grep delete | awk 'kill -9 {print "kill -9 " $2 }' > kill-deleted-pid.sh
chmod +x kill-deleted-pid.sh
./kill-deleted-pid.sh
```



历史case:

1.进程输出日志到磁盘文件中，打满磁盘，删除文件，此时磁盘空间仍未释放

2.业务php、nginx进程输出日志到磁盘中，同时logrotate采用Create切割日志和定时删除日志，出现df -h和du -sh \*显示不一致的情况，定位是有句柄未释放，继续向已被删除的日志写入数据

问题修复：

推荐业务使用CopyTruncate模式

《logrotate机制详解》[https://blog.51cto.com/u\_15501087/5834179 ](https://blog.51cto.com/u\_15501087/5834179)

