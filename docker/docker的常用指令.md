# Docker 的常用指令

```bash
docker search java
docker pull java
docker images
docker save docker.io/java > /root/java.tar.gz
docker rmi docker.io/java
docker ps -a # 查看容器运行状态
```

容器生命周期管理

```bash
docker [run|start|stop|restart|kill|rm|pause|unpause]
```

容器操作运维

```bash
docker [ps|inspect|top|attach|events|logs|wait|export|port]
```

容器rootfs命令


```bash
docker [commit|cp|diff]
```

镜像仓库

```bash
docker [login|pull|push|search]
```

本地镜像管理

```bash
docker [images|rmi|tag|build|history|save|import]
```

其他命令

```bash
docker [info|version]
```

## 启动

-it : 以交互式启动容器
-name : 指定容器别名

```bash
docker run -it --name myjava docker.io/java
```

## 映射

### 端口映射

-p 9001:8080

9000:宿主机9000端口

8085:docker 虚拟主机8085端口

把 docker 虚拟主机的8085端口映射到宿主机的9000端口上

### 文件映射

-v /root/project:/soft --privileged

/root/project : 宿主机 /root/project 目录
/soft : docker 虚拟主机 /soft 目录
--privileged : 给予最高权限

```bash
docker run -it --name myjava -p 9000:8080 -p 9001:8085 -v /root/project:/soft --privileged docker.io/java bash
```

## 暂停和停止

```bash
docker pause myjava #暂停
docker unpause myjava #恢复
docker stop myjava #停止
docker start -i myjava #启动
```