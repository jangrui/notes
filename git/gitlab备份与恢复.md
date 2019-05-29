# gitlab 备份与恢复

## 备份

使用Gitlab一键安装包安装Gitlab非常简单, 同样的备份恢复与迁移也非常简单. 使用一条命令即可创建完整的Gitlab备份

### 手动备份

```bash
gitlab-rake gitlab:backup:create CRON=1
```

### 自动备份

```bash
tee /etc/crontab <<-end
0 2 * * *  /opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1
end
systemctl restart crond
```

### docker 环境中备份

```bash
docker exec -it igitlab gitlab-rake gitlab:backup:create CRON=1
```

## 恢复

- 查看备份

```bash
gitlab-rake gitlab:backup:restore
```

- 停止数据连接

```bash
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```

- 验证是否全部停止

```bash
gitlab-ctl status
```

- 恢复数据

```bash
gitlab-rake gitlab:backup:restore BACKUP=1559125666_2019_05_29_11.11.0
gitlab-ctl reconfigure
gitlab-ctl restart
```

- 检查运行情况

```bash
gitlab-rake gitlab:check SANITIZE=true
```

### docker 环境中恢复

```bash
docker exec -it igitlab gitlab-ctl stop unicorn
docker exec -it igitlab gitlab-ctl stop sidekiq
docker exec -it igitlab gitlab-rake gitlab:backup:restore BACKUP=1559125666_2019_05_29_11.11.0
docker exec -it igitlab gitlab-ctl reconfigure
docker exec -it igitlab gitlab-ctl restart
```