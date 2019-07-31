# 给 gitlab 添加 SMTP

## 修改配置文件

编辑/etc/gitlab/gitlab.rb文件（加到文件最后面就好了）。

以QQ企业邮箱为例：

```bash
gitlab_rails['smtp_enable'] = true

gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"

gitlab_rails['smtp_port'] = 465

gitlab_rails['smtp_user_name'] = "gitlab@xxx.com"

gitlab_rails['smtp_password'] = "******"

gitlab_rails['smtp_authentication'] = "login"

gitlab_rails['smtp_enable_starttls_auto'] = true

gitlab_rails['smtp_tls'] = true

gitlab_rails['gitlab_email_from'] = 'gitlab@xxx.com'
```

有的教程可能会说去改/opt/gitlab/etc/gitlab.rb，是错的，一切以官网文档为准

## 重新配置gitlab

```bash
gitlab-ctl reconfigure
```

- 通过命令行测试邮件是否发送成功（也可以不测）

```bash
gitlab-rails console
irb(main):003:0> Notify.test_email('xxx@qq.com', 'Message Subject', 'Message Body').deliver_now
```

来自:

[配置gitlab通过smtp发送邮件](https://www.centos.bz/2017/08/gitlab-send-email-with-smtp/)
