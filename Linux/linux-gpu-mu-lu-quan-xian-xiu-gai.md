---
description: 修复GPU监控丢失问题
---

# Linux GPU目录权限修改

```sh
# 1.遍历node列表 node为ip
kubeconfig=$1

for i in `cat node_ips`
do
    echo $i
    # 创建文件用于记录ip处理结果
    touch $i
    # 2.远程ssh免密登陆机器执行chmod命令
    chmod 777 /baidu-cgpu/mps/
    chmod 777 /baidu-cgpu/nvidia*
    # 3.获取机器上的cgpu-monitor并判断重建成功，强制删除
    
    # 4.远程ssh免密登陆机器执行ln -s命令
    cd /home/opt/cgpu/lib64/
    ln -s libcgpu.so.2.1.15 libnvidia-ml.so.1
    ln -s  libnvidia-ml.so.1  libnvidia-ml.so
    
    # huo qu
      
done
```
