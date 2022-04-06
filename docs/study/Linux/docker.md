# 安装

安装参考https://developer.aliyun.com/article/110806

 1.卸载旧版本

```shell
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```

2. Install the `yum-utils` package (which provides the `yum-config-manager` utility) and set up the **stable** repository.

```shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息  
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast 
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start
```

补充相关细节

```shell
# yum-utils 是管理repository及扩展包的工具,包含一系列yum工具，解决软件依赖之利器！提供了 yum-config-manager 工具（用于添加yum源）
# device-mapper-persistent-data lvm2 这两个是Device Mapper需要的(docker官网没有安装这个)
# docker-ce.repo是docker软件源信息文件，就一个配置文件
# sudo yum makecache fast  # 软件包信息提前在本地缓存一份，用来提高搜索安装软件的速度
# 安装完可以运行 docker run hello-world 检查安装是否成功)(会自动拉一个镜像)
```

参考 https://blog.csdn.net/The_Time_Runner/article/details/104481294



> 配置镜像源

参考 https://yeasy.gitbook.io/docker_practice/

vim  /etc/docker/daemon.json

```
# 内容如下：
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}

# 退出并保存
:wq

# 使配置生效
systemctl daemon-reload

# 重启Docker
systemctl restart docker
```





# 基本命令



## 镜像

```shell
docker images # 列出已有镜像  docker images

docker search [想要查找的镜像]  # docker search mysql

docker pull [想要下拉镜像] # docker pull mysql:8

docker rmi [想要删除的镜像id] # docker rmi feb5d9fea6a5
	docker rmi -f $(docker images -aq)
```



## 容器

```shell
docker run [可选参数] image
# 参数说明
--name=“Name”  容器名字  tomcat01  tomcat02  用来区分容器
-d		后台方式运行
-it		使用交互方式运行，进入容器查看内容
-p		指定容器的端口		-p 8080:8080
	-p	ip:主机端口：容器端口
	-p	主机端口：容器端口（常用）
	-p	容器端口
	容器端口
-P		随机指定端口


docker rm [想要删除的容器id]  # 删除容器
	docker rm -f [想要删除的容器id]  # 强制删除容器
	docker rm -f $(docker ps -aq)		# 删除所有容器
	docker ps -a -q|xargs docker rm -f	# 删除所有的容器


docker start 容器id			# 启动容器
docker restart 容器id			# 重启容器
docker stop 容器id			# 停止当前正在运行的容器
docker kill 容器id			# 强制停止当前的容器

exit 			# 直接退出容器并关闭
Ctrl + P + Q	# 容器不关闭退出

docker attach [容器id]  # 连接进容器的主进程， 不会新开进程
docker exec -it [容器id] bash # 连接进容器中，并新开一个bash终端（新开进程）
docker top [容器id] # 显示容器运行着的进程, 注意是运行着的进程！
docker logs -tf --tail [尾部行数] [容器id] # 查看容器运行的日志
docker cp [源] [目的]  # 容器和宿主机之前相互复制文件
	docker cp a.txt 32fc06bf21e6:/tmp  # 将宿主机的a.txt复制到容器32fc06bf21e6的/tmp下
	docker cp 32fc06bf21e6:/tmp/b.txt ./  # 将容器32fc06bf21e6的/tmp/b.txt复制到宿主机的当前目录./下
```





## 网络





## 其他命令

```shell
docker version # docker版本信息
docker info # 系统级别的信息，包括镜像和容器的数量, docker镜像源
docker 具体命令 --help # 比如 docker run --help
docker inspect [容器id或者镜像id] # 查看容器或者镜像的元信息
```



搜索镜像历史版本

```shell
# centos的历史版本
curl https://registry.hub.docker.com/v1/repositories/centos/tags\
| tr -d '[\[\]" ]' | tr '}' '\n'\
| awk -F: -v image='centos' '{if(NR!=NF && $3 != ""){printf("%s:%s\n",image,$3)}}'

# mysql的历史版本
curl https://registry.hub.docker.com/v1/repositories/mysql/tags\
| tr -d '[\[\]" ]' | tr '}' '\n'\
| awk -F: -v image='mysql' '{if(NR!=NF && $3 != ""){printf("%s:%s\n",image,$3)}}'
```







# 其他

> 部署Demo (Nginx, Tomcat, ES+kibana)

部署Nginx

```shell
docker run -d --name nginx01 -p 3344:80 nginx	# 后台方式启动启动镜像 （注意要防火墙要开启3344端口）
# 参数说明
# -d 后台运行
# -name 给容器命名
# -p 宿主机端口：容器内部端口

curl localhost:3344	# 本地访问测试
```



部署Tomcat

部署ES+kibana





镜像配置目录： /etc/docker/daemon.json

默认工作路径： /var/lib/docker/

