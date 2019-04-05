# pipenv

## pip 国内源

阿里云 http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 

豆瓣(douban) http://pypi.douban.com/simple/ 

清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/

中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

```bash
pip install web.py -i http://pypi.douban.com/simple
```

如果想配置成默认的源，方法如下：

需要创建或修改配置文件（一般都是创建）

linux的文件在 `~/.pip/pip.conf `

windows在%HOMEPATH%\pip\pip.ini

修改内容为：

```bash
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

## 安装pip

```bash
$ pip install pipenv
```

## 用法

在使用pipenv之前，必须彻底的忘记pip这个东西

新建一个运行虚拟环境的目录，并进入该文件夹：

- pipenv --three 				# 会使用当前系统的Python3创建环境
- pipenv --python 3.6 			# 指定某一Python版本创建环境
- pipenv shell 					# 激活虚拟环境
- pipenv --where  				# 显示目录信息
- pipenv --venv  				# 显示虚拟环境信息
- pipenv --py  					# 显示Python解释器信息
- pipenv install requests 		# 安装相关模块并加入到Pipfile
- pipenv install django==1.11 	# 安装固定版本模块并加入到Pipfile
- pipenv graph 					# 查看目前安装的库及其依赖
- pipenv check 					# 检查安全漏洞
- pipenv uninstall --all  		# 卸载全部包并从Pipfile中移除

## 设置国内源：

修改Pipfile文件中[source]下面url属性


`url = "https://pypi.tuna.tsinghua.edu.cn/simple"`
