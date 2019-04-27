# sed

- `-x` 显示执行命令
- `-e` 只要发生错误，就停止执行
- `-u` 遇到变量不存在，就停止执行
- `-o pipefail` 只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。

set命令的上面这四个参数，一般都放在一起使用。

```bash
# 写法一
set -euxo pipefail

# 写法二
set -eux
set -o pipefail
```

这两种写法建议放在所有 Bash 脚本的头部。

另一种办法是在执行 Bash 脚本的时候，从命令行传入这些参数。

```bash
$ bash -euxo pipefail script.sh
```

## 参考：

> [Bash 脚本 set 命令教程](http://www.ruanyifeng.com/blog/2017/11/bash-set.html)