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

1\.

