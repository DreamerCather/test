---
coverY: 0
---

# Linux删除文件磁盘空间未释放

&#x20;      在Linux环境中，很容易出现一种情况，文件删除了但是磁盘空间未释放，句柄泄漏，导致磁盘空间打满，因此文档主要看下如何定位该问题以及针对本次case如何修复



问题定位：

1.在机器上执行 &#x20;

{% code title="" %}
```sh
lsof -n |grep deleted
```
{% endcode %}





问题修复：



历史case:

1\.
