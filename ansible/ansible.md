# Ansible

## 安装

- CentOS

```bash
yum install -y ansible
```
- Ubuntu
  
```bash
apt-get install -y software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install -y ansible
```

- MacOS

```bash
brew install ansible
```

## Ansible 配置文件路径

- export ANSIBLE_CONFIG
- ./ansible.cfg
- ~/.ansible.cfg
- /etc/ansible.cfg

## 官方配置模板

- [github: ansible.cfg](https://raw.github.com/ansible/ansible/devel/examples/ansible.cfg)
