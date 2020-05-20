<!--
 * @Author: jangrui
 * @Date: 2019-09-19 21:41:04
 * @LastEditors: jangrui
 * @LastEditTime: 2019-10-16 23:31:45
 * @Version: 
 * @Description: npm
 -->

# npm

## 安装

- mac

```bash
brew install node
```

- centos

```bash
yum install -y epel-release && yum makecache fast
yum install -y nodejs
npm i -g n && n latest
npm i -g npm
```

- ubuntu

```bash
apt-get install nodejs
apt-get install npm
```

## 镜像源

- 淘宝源

```bash
npm config set registry http://registry.npm.taobao.org/
```

- 官方

```bash
npm config set registry https://registry.npmjs.org/
```

## 更新

- npm 更新

```bash
npm i -g npm
```

- node 更新

```bash
npm i -g n
n latest
```
