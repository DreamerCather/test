---
description: 修复GPU监控丢失问题 10.95.9.143
---

# Linux GPU目录权限修改

```sh
# 1.遍历node列表 node为ip
kubeconfig=$1

for i in `cat node_ips`
do
    echo $1 
    
    # 创建文件用于记录ip处理结果
    touch $i
    
    # 2.远程ssh免密登陆机器执行chmod命令
    echo $i | xargs -i{} ssh -o StrictHostKeyChecking=no zhaomingfu@{} "sudo chmod 777 /baidu-cgpu/mps/ && sudo chmod 777 /baidu-cgpu/nvidia*"
 
    # 3.获取机器上的cgpu-monitor并判断重建成功，强制删除，没有pod则无需删除
    podName=`kubectl --kubeconfig $1 -n kube-system get pods --field-selector spec.nodeName=$i | grep baidu-cgpu-monitor-lrs | awk '{print $1}'`
    echo "cgpu-monitor-origin: $podName" > $i
    kubectl $1 --kubeconfig -n kube-system delete pods $podName --force --grace-period=0
    sleep 8
    for i in {1..100};
    do
       newPodName=`kubectl --kubeconfig $1 -n kube-system get pods --field-selector spec.nodeName=$i | grep baidu-cgpu-monitor-lrs | awk '{print $1}'`
       if [[ $newPodName != $podName ]];
       then
          echo "cgpu-monitor-new: $newPodName" > $i
          break
       fi  
    done
    
    # 4.远程ssh免密登陆机器执行ln -s命令
    echo $i | xargs -i{} ssh -o StrictHostKeyChecking=no zhaomingfu@{} "sudo cd /home/opt/cgpu/lib64/ && ln -s libcgpu.so.2.1.15 libnvidia-ml.so.1 && ln -s  libnvidia-ml.so.1  libnvidia-ml.so"
    
    # 5.找出GPU Pod进行相关命令执行
    for j in podList1
    do
       isGpu=kubectl --kubeconfig $1 -n predictor get pods $i -ojson | jq -c '.metadata.annotations.BAIDU_COM_GPU_ASSIGNED'
       if [[ $isGpu == 'true' ]];
       then
          kubectl --kubeconfig $1 -n predictor exec -it $j nvidia-smi
       fi  
    done
    
    podList2=`kk --kubeconfig $1 get pods -n ecom-feedads-lingli-feedads-predictor --field-selector spec.nodeName=$i`
    for j in podList2
    do
       isGpu=kubectl --kubeconfig $1 -n ecom-feedads-lingli-feedads-predictor get pods $i -ojson | jq -c '.metadata.annotations.BAIDU_COM_GPU_ASSIGNED'
       if [[ $isGpu == 'true' ]];
       then
          kubectl --kubeconfig $1 -n ecom-feedads-lingli-feedads-predictor exec -it $j nvidia-smi
       fi  
    done
    
    # 6. 远程ssh免密登陆机器执行删除软链命令
    echo $i | xargs -i{} ssh -o StrictHostKeyChecking=no zhaomingfu@{} "sudo cd /home/opt/cgpu/lib64/ && rm -rf libnvidia-ml.so && rm -rf libnvidia-ml.so.1"
      
done
```
