# zsh + oh-my-zsh + agnoster

## 安装 zsh

- Mac 

```bash
brew install -y zsh
```

- Ubuntu

```bash
apt-get install -y zsh
```

- Centos

```bash
yum install -y zsh
```

切换为 zsh

```bash
chsh -s /bin/zsh
```

重启终端生效

## 安装 oh-my-zsh

```bash
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 切换 agnoster

```bash
sed -i '' 's/^ZSH.*/ZSH_THEME="agnoster"/' ~/.zshrc
source ~/.zshrc
```

## 命令补全插件

- MacOS

```bash
brew install zsh-completions
```

- Ubuntu

```bash
apt-get install -y zsh-completions
```

- CentOS

```bash
yum install -y zsh-completions
```

### 配置 zshrc

```bash
sed -i '' 's/^plugins.*/plugins=(zsh-completions git)/' ~/.zshrc
echo 'autoload -U compinit && compinit' >> ~/.zshrc
source ~/.zshrc
```

## 自动补全插件

- MacOS

```bash
brew install zsh-autosuggestions
echo 'source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh' >> ~/.zshrc
source ~/.zshrc
```

- Linux

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
sed -i '' 's/^plugins.*/plugins=(zsh-autosuggestions zsh-completions git)/' ~/.zshrc
source ~/.zshrc
```