# ansible 常用模块

## File

在目标主机创建文件或目录，并指定其系统权限

```bash
- name: 创建一个文件
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
```

## Copy

实现 ansible 服务端到目标主机的文件传送

```bash
- name: 
  copy: 'remote_src=no src=roles/testbox/file/foo.sh dest=/root/foo.sh mode=0644 force=yes'
```

> - force  # 强制执行

## Stat

获取远程文件状态信息

```bash
- name: 检查 foo.sh 是否存在
  stat: 'path=/root/foo.sh'
  register: script_stat
```

> - register # 变量赋值

## Debug

打印语句到 ansible 执行输出

```bash
- debug: msg=foo.sh exists
  when: script_stat.exists
```

> - when # 条件判断，正确执行，不正确不执行

## Command

用来执行 Linux 目标主机命令行

```bash
- name: 执行一个命令
  command: "sh /root/foo.sh"
```

## Shell

用来执行 Linux 目标主机命令行

```bash
- name: 执行 shell 语句
  shell: "echo 'test' >> test.log"
```

## Template

实现 ansible 服务端到目标主机的 jinja2 模板传送

```bash
- name: 编写一个 nginx 配置文件
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

## Packaging

调用目标主机系统包管理工具（yum，apt）进行安装软件包

```bash
- name: yum 安装 nginx 最新版
  yum: pkg=nginx state=latest

- name: apt 安装 nignx 最新版
  apt: pkg=nginx state=latest
```

## Service

调用 service/systemctl 管理目标主机系统服务

```bash
- name: 启动 nignx 服务
  service: name=nginx state=started
```
## yum_repository

在基于RPM的Linux发行版中添加或删除YUM源。

```bash
- name: Add repository
  yum_repository:
    name: epel
    description: EPEL YUM repo
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: Add multiple repositories into the same file (1/2)
  yum_repository:
    name: epel
    description: EPEL YUM repo
    file: external_repos
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
    gpgcheck: no

- name: Add multiple repositories into the same file (2/2)
  yum_repository:
    name: rpmforge
    description: RPMforge YUM repo
    file: external_repos
    baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
    mirrorlist: http://mirrorlist.repoforge.org/el7/mirrors-rpmforge
    enabled: no

# Handler showing how to clean yum metadata cache
- name: yum-clean-metadata
  command: yum clean metadata
  args:
    warn: no

# Example removing a repository and cleaning up metadata cache
- name: Remove repository (and clean up left-over metadata)
  yum_repository:
    name: epel
    state: absent
  notify: yum-clean-metadata

- name: Remove repository from a specific repo file
  yum_repository:
    name: epel
    file: external_repos
    state: absent
```
