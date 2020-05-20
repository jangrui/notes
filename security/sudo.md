# sudo

## sudo 无需密码验证

```bash
[jangrui@localhost ~]$ sudo echo "%wheel  ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
```

## sudo 提示找不到该命令

```bash
sudo sed -i 's,env_reset,!&,g' /etc/sudoers
echo "alias sudo='sudo env PATH=$PATH'" >> ~/.bashrc
source ~/.bashrc
```
