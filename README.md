<!--
 * @Author: jangrui
 * @Date: 2019-07-02 21:40:50
 * @LastEditors: jangrui
 * @LastEditTime: 2019-08-26 20:43:36
 * @version: 
 * @Descripttion: Linux 运维学习路径
 -->

# Linux 运维学习路径

<!-- ![Linux](./_media/linux.png "linux.png") -->

- Linux
  - [Command](command/#command)
  - Shell
    - [Shell 特殊变量](shell/Shell特殊变量)
    - [shell 变量替换](shell/shell变量替换)
    - [字符串处理](shell/字符串处理)

- Service
  - [RAID 磁盘阵列技术](service/raid)
  - [LVM 逻辑卷管理器](service/lvm)
  - [SSH 远程管理主机服务](service/ssh)
  - [FTP 部署文件传输服务](service/ftp)
  - [Samba 部署文件共享服务](service/samba)
  - [NFS 部署文件共享服务](service/nfs)
  - [AutoFs 部署自动挂载服务](service/autofs)
  - [Bind 部署域名解析服务](service/bind)
  - [DHCP 部署动态分配主机地址](service/dhcp)
  - [Postfix + dovecot 部署邮件服务](service/mail)
  - [Squid 部署代理服务](service/proxy)
  - [ISCSI 部署网络存储服务](service/iscsi)
  - [PXE + Kickstart 部署无人值守安装服务](service/unattended)
  - [LDAP 部署轻量级目录服务](service/ldap)
  - [Rsync 部署同步服务](service/rsync)
  <!-- - [SSHFS](service/sshfs) -->
  <!-- - [OXFS](service/oxfs) -->

- Web
  - Nginx
    - [Nginx 文档](http://www.nginx.cn/doc)
    - [基本配置与参数说明](nginx/基本配置与参数说明)
    - [负载均衡](nginx/负载均衡)
  - Tomcat
  - Apache
  - WebLogic

- DB
  - Mysql
    - PXC
  - Oracle
  - Mangodb
  - Memcached
  - Redis

- ServiceLoadBalacing
  - LVS
  - Keepalive
  - HAProxy
  - Nginx

- DevOps
  - Jenkins
    - [安装](jenkins/install)
    - [pipline](jenkins/pipline)
    - [中文文档](https://jenkins.io/zh/doc/)
    - [Pipeline 详解](https://jenkins.io/zh/doc/book/pipeline/syntax/)

  - Ansible
    - [安装](ansible/install)
    - [模块](ansible/module)
    - [中文文档](http://www.ansible.com.cn/)
    - [Playbook 详解](http://www.ansible.com.cn/docs/playbooks.html)

  - Docker
    - [安装及加速](docker/docker安装及加速)
    - [常用指令](docker/docker的常用指令)
    - [Dockerfile 详解](docker/dockerfile)

  - Kubernetes
    - [Kubernetes 高可用集群部署](k8s/kubernetes-ha-kubeadm)
    - [Master 节点也做 Worker 节点](k8s/master-worker)

- Monitor
  - Zabbix

- Virtualization
  - KVM
  - Xen
  - Openstack
  - VMware

- Security
  - Firewall
  - iptables
  - SELinux
  - Kerberos
  - ACL
  - Jumpserver
    - [Jumpserver 跳板机](http://docs.jumpserver.org/zh/docs/index.html)
