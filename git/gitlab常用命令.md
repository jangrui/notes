# gitlab 常用命令

```bash
# 卸载
gitlab-ctl uninstall

# 查看版本
cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

# echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
# sysctl -p
# echo never > /sys/kernel/mm/transparent_hugepage/enabled

# 检查gitlab
gitlab-rake gitlab:check SANITIZE=true --trace
gitlab-rake gitlab:check
gitlab-rake gitlab:check SANITIZE=true

# 查看日志
gitlab-ctl tail

# 数据库关系升级
gitlab-rake db:migrate

# 清理缓存
gitlab-rake cache:clear

# 更新 gitlab 包
yum update gitlab-ce

# 升级 gitlab
yum install gitlab-ce

# 升级数据命令
gitlab-ctl pg-upgrade
```

## 服务管理

```bash
# 启动所有 gitlab 组件：
gitlab-ctl start

# 停止所有 gitlab 组件：
gitlab-ctl stop

# 停止所有 gitlab postgresql 组件：
gitlab-ctl stop postgresql

# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq

# 重启所有 gitlab 组件：
gitlab-ctl restart

# 重启所有 gitlab-workhorse 组件：
gitlab-ctl restart gitlab-workhorse

# 查看服务状态
gitlab-ctl status

# 生成配置启动服务
gitlab-ctl reconfigure
```

## 日志查看

```bash
# 查看日志
gitlab-ctl tail

# 检查 redis 的日志
gitlab-ctl tail redis

# 检查 postgresql 的日志
gitlab-ctl tail postgresql

# 检查 gitlab-workhorse 的日志
gitlab-ctl tail gitlab-workhorse

# 检查 logrotate 的日志
gitlab-ctl tail logrotate

# 检查 nginx 的日志
gitlab-ctl tail nginx

# 检查 sidekiq 的日志
gitlab-ctl tail sidekiq

# 检查 unicorn 的日志
gitlab-ctl tail unicorn
```

## 重置管理员密码

```bash
# 使用 rails 工具打开终端
gitlab-rails console production

# 查询用户的 email，用户名，密码等信息，id:1 表示 root 账号
user = User.where(id: 1).first  

#重新设置密码
user.password = '新密码'
user.password_confirmation = '新密码'

# 保存密码
user.save!

# 完整的操作 ruby 脚本
user = User.where(id: 1).first
user.password = '新密码'
user.password_confirmation = '新密码'
user.save!
```

## 备份

```bash
# 修改/etc/gitlab/gitlab.rb来修改默认存放备份文件的目录:
tee /etc/gitlab/gitlab.rb <<-end
gitlab_rails['backup_path'] = "/mnt/backups"
end
gitlab-ctl reconfigure
gitlab-ctl restart

# 备份
gitlab-rake gitlab:backup:create

#通过crontab使用备份命令实现自动备份
crontab -e  
# 每天2点备份gitlab数据
0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create
0 2 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create

#备份保留七天
#设置只保存最近7天的备份，编辑 /etc/gitlab/gitlab.rb 配置文件，找到如下代码，删除注释 # 保存

# /etc/gitlab/gitlab.rb 配置文件 修改下面这一行
gitlab_rails['backup_keep_time'] = 604800
```

## 恢复

```bash
# 停止相关数据连接服务
gitlab-ctl stop unicorn
# ok: down: unicorn: 0s, normally up
gitlab-ctl stop sidekiq
# ok: down: sidekiq: 0s, normally up

# 从xxxxx编号备份中恢复，1406691018为备份文件的时间戳
gitlab-rake gitlab:backup:restore BACKUP=1406691018
```