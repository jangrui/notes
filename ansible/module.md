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

## yum_repository

- 在基于RPM的Linux发行版中添加或删除YUM源。
- 如果要更新现有源，请改用ini_file。

```bash
- name:  添加源
  yum_repository:
    name: epel
    description: EPEL YUM repo
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: 将多个源添加到同一个文件中 (1/2)
  yum_repository:
    name: epel
    description: EPEL YUM repo
    file: external_repos
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
    gpgcheck: no

- name: 将多个源添加到同一个文件中 (2/2)
  yum_repository:
    name: rpmforge
    description: RPMforge YUM repo
    file: external_repos
    baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
    mirrorlist: http://mirrorlist.repoforge.org/el7/mirrors-rpmforge
    enabled: no

# Handler showing how to clean yum metadata cache
- name: 清理缓存
  command: yum clean metadata
  args:
    warn: no

# Example removing a repository and cleaning up metadata cache
- name: 删除源并清理缓存
  yum_repository:
    name: epel
    state: absent
  notify: yum-clean-metadata

- name: 制定删除repo文件
  yum_repository:
    name: epel
    file: external_repos
    state: absent
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

- 控制远程主机上的服务。支持的init系统包括BSD init，OpenRC，SysV，Solaris SMF，systemd，upstart。
- 对于Windows目标，请改用win_service模块。

```bash
- name: 启动 httpd
  service:
    name: httpd
    state: started

- name: 停止 httpd
  service:
    name: httpd
    state: stopped

- name: 重启 httpd
  service:
    name: httpd
    state: restarted

- name: 重载 httpd
  service:
    name: httpd
    state: reloaded

- name: 开机启动 httpd
  service:
    name: httpd
    enabled: yes

- name: 基于进程启动, 启动 /usr/bin/foo 
  service:
    name: foo
    pattern: /usr/bin/foo
    state: started

- name: 带参数, 重启 etho 网卡
  service:
    name: network
    state: restarted
    args: eth0
```

## systemd

控制远程主机上的systemd服务。

> 由systemd管理的系统。

```bash
- name: 确保服务正在运行
  systemd:
    state: started
    name: httpd

- name: 在 debian 上停止 cron 服务
  systemd:
    name: cron
    state: stopped

- name: 重启 crond 服务前先重载配置
  systemd:
    state: restarted
    daemon_reload: yes
    name: crond

- name: 重载 httpd
  systemd:
    name: httpd
    state: reloaded

- name: 开机启动 httpd, 并确保没有被锁定
  systemd:
    name: httpd
    enabled: yes
    masked: no

- name: 启动 dnf-automatic.timer 并开机启动
  systemd:
    name: dnf-automatic.timer
    state: started
    enabled: yes

- name: 强制重载配置 (2.4及以上版本)
  systemd:
    daemon_reload: yes
```

