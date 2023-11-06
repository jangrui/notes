
```bash
#在master节点之外的节点进行操作
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
##重启kubelet
systemctl restart kubelet
##重启docker
systemctl restart docker
yum remove -y kubeadm kubectl kubelet
rm -rf ~/.kube
docker stop `docker ps -qa`
docker rm `docker ps -qa`
yum remove -y docker*

fuser -mv  /var/lib/ceph/osd/ceph-1
kill -9 1234
rm -rf /data/*
umount /data/
```