# GitLab 安装

为不了不引起一些不必要的麻烦，关闭了 firewall 防火墙及 selinux,并重启系统。

```bash
systemctl stop firewalld
systemctl disable firewalld
sed -i 's/=enforcing/=disable/g' /etc/selinux/config
reboot
```

## 下载 gitlab-ce 源

```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

## 安装 gitlab-ce

```bash
yum install -y gitlab-ce
```

## docker 部署 gitlab-ce

```bash
# 官方镜像
docker pull gitlab/gitlab-ce:latest
# 社区中文版
docker pull beginor/gitlab-ce:latest
```

启动之前先创建几个所需目录，毕竟数据至上：

```bash
mkdir -p /etc/gitlab/ssl
mkdir -p /var/log/gitlab
mkdir -p /var/opt/gitlab
docker run -it -d --name gitlab \
    --hostname gitlab.jangrui.com \
    -p 443:443 -p 22:22 \
    -v /etc/gitlab:/etc/gitlab \
    -v /var/opt/gitlab:/var/opt/gitlab \
    -v /var/log/gitlab:/var/log/gitlab \
    --restart unless-stopped \
    gitlab/gitlab-ce:latest
```

## 配置 域名、ssl、smtp

```bash
tee /etc/gitlab/gitlab.rb <<-'END'
external_url 'https://gitlab.jangrui.com'
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git.jangrui.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git.jangrui.com.key"
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "xxxx@qq.com"
gitlab_rails['smtp_password'] = "xxxxpassword"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = "xxxx@qq.com"
# 设置只保存最近7天的备份
gitlab_rails['backup_keep_time'] = 604800
user["git_user_email"] = "xxxx@qq.com"
END
docker exec -it gitlab gitlab-ctl reconfigure
```

> 参考: [https://docs.gitlab.com/omnibus/docker/README.html](https://docs.gitlab.com/omnibus/docker/README.html)
