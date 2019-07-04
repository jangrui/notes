# master同时也做worker节点

使用kubeadm初始化的集群，出于安全考虑 Pod 不会被调度到 Master 上，也就是说 Master 不参与工作负载。

也就是说, 只有一台主机的情况下, 我们无法启动pod, 但有的时候我们的确只有一台机器, 这个时候可以执行命令, 使Master Node参与工作负载。

1. 允许 master 节点调度 pod

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

输出如下：

```bash
node/m1 untainted
node/m2 untainted
node/m3 untainted
error: taint "node-role.kubernetes.io/master:" not found
```

2. 禁止 master 节点调度 pod

```bash
kebectl taint nodes m1 node-role.kubernetes.io/master=true:NoSchedule
```

输出如下：

```bash
node/m1 tainted
```