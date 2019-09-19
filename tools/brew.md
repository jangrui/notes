<!--
 * @Author: jangrui
 * @Date: 2019-07-02 21:40:50
 * @LastEditors: jangrui
 * @LastEditTime: 2019-09-19 21:23:10
 * @version: 
 * @Descripttion: zsh
 -->

# brew

## 安装

- Homebrew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- Homebrew cask

```bash
brew tap caskroom/cask
```

## 镜像源

- 清华大学

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
brew update
```

- 阿里云

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
brew update
```

- 官方

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git
brew update
```
