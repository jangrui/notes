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
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 切换 agnoster

```bash
sed -i 's/^ZSH.*/ZSH_THEME="agnoster"/' ~/.zshrc
source ~/.zshrc
```

> MacOS sed 报错

```bash
sed -i '' 's/^plugins.*/plugins=(zsh-completions git)/' ~/.zshrc
```

## 命令补全插件

```bash
git clone https://github.com/zsh-users/zsh-completions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions
sed -i 's/^plugins.*/plugins=(zsh-completions git)/' ~/.zshrc
echo 'autoload -U compinit && compinit' >> ~/.zshrc
source ~/.zshrc
```

## 自动补全插件

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
sed -i 's/^plugins.*/plugins=(zsh-autosuggestions zsh-completions git)/' ~/.zshrc
source ~/.zshrc
```

## 高亮插件

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
sed -i 's/^plugins.*/plugins=(zsh-syntax-highlighting zsh-autosuggestions zsh-completions git)/' ~/.zshrc
source ~/.zshrc
```
