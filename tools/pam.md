# PAM 身份认证

## 使用 pam 只允许 wheel 组成员切换 root

```bash
[root@localhost ~]# cat /etc/pam.d/su
#%PAM-1.0
auth		sufficient	pam_rootok.so
# Uncomment the following line to implicitly trust users in the "wheel" group.
#auth		sufficient	pam_wheel.so trust use_uid
# Uncomment the following line to require a user to be in the "wheel" group.
auth		required	pam_wheel.so use_uid
...
```

找到上面两行，去掉行首的“#”

修改只允许 wheel 组切换 root 

```bash
echo "US_WHEEL_ONLY yes" >> /etc/login.defs
```

加入用户到 wheel 组

```bash
usermod -G wheel jangrui
```

### 效果

```bash
[jangrui@localhost ~]$ sudo useradd wpeng
[sudo] jangrui 的密码：
[jangrui@localhost ~]$ sudo passwd wpeng
更改用户 wpeng 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[jangrui@localhost ~]$ su - wpeng
密码：
[wpeng@localhost ~]$ su -
密码：
su: 拒绝权限
[wpeng@localhost ~]$ exit
登出
[jangrui@localhost ~]$ su -
密码：
上一次登录：二 3月 19 16:52:10 CST 2019pts/0 上
最后一次失败的登录：二 3月 19 16:54:03 CST 2019pts/0 上
最有一次成功登录后有 1 次失败的登录尝试。
[root@localhost ~]# 
```