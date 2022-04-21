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
	
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名：tag
	# 例如 docker commit -a="hwj" -m="add webapps app" d798a5946c1f tomcat-hwj:1.0

docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG] # 增加容器标签 （由此可实现改镜像名）
	# 例如 docker tag tomcat-hwj:1.0 tomcat-hwj:latest
	
docker history [镜像名or镜像id] # 查看镜像的提交层
	# 例如 docker history mysql:5.7
	
docker save # 保存成tar包
	# docker save -o ./tomcat-hwj.tar tomcat-hwj:1.0  # -o output
	
docker load # 载入tar包
	# docker load -q -i tomcat-hwj.tar # -q镇压载入时的输出， -i input
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
--restart=always 随着docker服务的重启而重启
--privileged=true 优先启动
--rm 容器用完就删

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



## 数据卷

```shell
docker volume ls  # 数据卷一般都是在/var/lib/docker/volumes/[数据据卷名]/_data目录里面
docker run -d -P --name nginx01 -v /etc/nginx nginx  # 匿名挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx # 具名挂载，自动创建一个叫juming-nginxjuming-nginx的数据卷
	# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载！
    -v	容器内路径					# 匿名挂载
    -v	卷名:容器内路径			   # 具名挂载
    -v /主机路径:容器内路径			  # 指定路径挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx 
	# :ro read only
	# :rw read write
	
docker run --volumes-from [某容器名或容器id] #使用某容器的数据卷作为自己的数据卷， 实现容器之间数据共享
$ docker run -d -p 3344:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
$ docker run -d -p 3345:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7
```





## 网络

```shell
docker network ls 
docker network inspect [网络id] # docker network inspect ac8239bb3527
docker network rm
docker network create
docker network connect
```





## 其他命令

```shell
docker version # docker版本信息
docker info # 系统级别的信息，包括镜像和容器的数量, docker镜像源
docker 具体命令 --help # 比如 docker run --help
docker inspect [容器id或者镜像id] # 查看容器或者镜像的元信息
docker stats  # 查看容器状态，包括CPU使用率和MEM使用率
docker prune # 命令用来删除不再使用的 docker 对象。
	docker image prune -a # 删除所有未被容器使用的镜像
	docker container prune # 删除所有停止运行的容器
	docker volume prune # 删除所有未被挂载的卷
	docker network prune # 删除所有未被使用的网络:
	docker system prune # 删除 docker 所有资源:
docker login # 登录
docker push # 发布镜像到DockerHub或者阿里云镜像仓库
docker build # 需要写Dockerfile文件
	docker build -f dockerfile1 -t my-centos:1.0 .  # 
	docker build -t my-centos:1.0 .  # 当文件为Dockerfile， 可以省略-f Dockerfile
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







# 部署

## 部署Nginx

```shell
docker run -d --name nginx01 -p 3344:80 nginx	# 后台方式启动启动镜像 （注意要防火墙要开启3344端口）
# 参数说明
# -d 后台运行
# -name 给容器命名
# -p 宿主机端口：容器内部端口

curl localhost:3344	# 本地访问测试
```



## 部署Tomcat

```shell
docker run -d -p 3355:8080 --name tomcat01 tomcat:9
```



## 部署ES

```shell
docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m"  elasticsearch:7.6.2

docker stats  # 查看容器状态，CPU使用率和MEM使用率
# 参数说明
# -e 是设置环境变量
# ES_JAVA_OPTS="-Xms64m -Xmx512m" 将jvm的内存限制在64~512M
```



## 部署Docker可视化界面 portainer

```shell
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```



## 部署Mysql

```shell
docker run -d -p 3355:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 用户名root
# 密码123456
# 端口3355
```





# Dockerfile

Dockerfile hello-world

```shell
# 创建一个dockerfile文件， 名字可以随机
vim dockerfile1


# 文件的内容 指定（大写） 参数,  这里的每一个命令都是镜像的一层！
​```
FROM centos

VOLUME ["volume01", "volume02"]

CMD echo "----end----"
CMD /bin/bash
​```

:wq退出

docker build -f dockerfile1 -t my-centos:1.0 .  # 在"."，即当前目录下生成镜像
# -t 指定标签

docker run -it [镜像id] bash  # 进入容器
```

简化版hello-world

```shell

vim Dockerfile  # 默认

# 输入内容
​```
FROM centos

VOLUME ["volume01", "volume02"]

CMD echo "----end----"
CMD /bin/bash
​```

:wq退出

docker build -t my-centos:2.0 . # 如果名字是Dockerfile，可以省略

docker run -it [镜像id] bash   # 进入容器
```



## Dockerfile命令

```shell
FROM			# 基础镜像，一切从这里开始构建
MAINTAINER		# 镜像是谁写的？ 姓名+邮箱
RUN				# 镜像构建的时候需要运行的命令
WORKDIR			# 镜像的工作目录
VOLUME			# 挂载的目录
EXPOSE			# 暴露的端口
ONBUILD			# 当构建一个被继承DockerFile 这个时候就会运行 ONBUILD 的指令，触发指令
ENV 			# 构建的时候设置环境变量！ 注意是构建环境时！
-------------------------------------
CMD				# 指定这个容器启动(运行)的时候要运行的命令，只有最后一个会生效可被替代
ENTRYPOINT		# 指定这个容器启动的时候要运行的命令， 可以追加命令
-------------------------------------
ADD				# 复制并自动解压
COPY			# 类似ADD, 将我们文件拷贝到镜像中
```



> Dockerfile 命令 Demo1

```shell
# 1. 编写Dockerfile的文件
[root@locahost /home/hwj]# cat mydockerfile-centos 
FROM centos:7
MAINTAINER hwj<123456789@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH		# 镜像的工作目录

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH  # 不生效
CMD echo "---end---"  # 不生效
CMD /bin/bash  # 只有最后一个CMD会生效

# 2. 通过这个文件构建镜像
# 命令 docker build -f dockerfile文件路径 -t 镜像名:[tag] .

[root@locahost /home/hwj]# docker build -f mydockerfile-centos -t mycentos:0.1 .

Successfully built d2d9f0ea8cb2
Successfully tagged mycentos:0.1
```



> Dockerfile 命令 Demo2

```shell
# 1. 编写Dockerfile的文件
FROM centos:7
MAINTAINER hwj<123456789@qq.com>

COPY readme.txt /usr/local/readme.txt

# 需要提前下载好这两个jar包到宿主机内
ADD jdk-8u73-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.37.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_73
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.37
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.37
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.37/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.37/bin/logs/catalina.out

# 2. 构建镜像
docker build -t diytomcat .

# 3. 测试
```



# 网络

在宿主机和容器里输入 ip addr 命令， 发现宿主机和容器的网卡是成对的，

这里使用的是linux的 **evth-pair** 技术: 就是一对虚拟设备接口，成对出现



> 容器互连 --link

```shell
docker run -dit --name centos01 centos:7 bash
docker run -dit --name centos02 --link centos01 centos:7 bash
	# 这样centos02里面可以直接ping centos01的名字了
	# 解释：相当于centos01变成了一个域名，不用知道centos01的ip就能ping通
docker exec -it [centos02的容器id] ping centos01
	# docker exec -it 581f361a9313 ping centos01
docker exec -it [centos02的容器id] cat /etc/hosts
	# docker exec -it 581f361a9313 cat /etc/hosts
```



## 网络模式

桥接

host

none

container



## 自定义网络

自定义网络

```shell
docker network create -d bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
	# -d, --driver string        Driver to manage the Network (default "bridge")
	# --subnet strings       Subnet in CIDR format that represents a network segment
	# --gateway strings      IPv4 or IPv6 Gateway for the master subnet
docker network inspect [网络id]  # 查看网络
```

使用自定义网络

```shell
docker run -dit --name centos01 --net mynet centos-hwj:7 bash   # 容器id：5886c0299c11
	# centos-hwj:7是在centos:7的基础上加net-tools工具
docker run -dit --name centos02 --net mynet centos-hwj:7 bash   # 容器id：b22d43ffe7d9
docker network inspect [网络id]  # 再次查看网络，会发现有两个容器连入了自定义网络
docker exec -it b22d43ffe7d9 ping centos01  # 发现可以ping通centos01


docker run -dit --name centos03 centos-hwj:7 bash  # 容器id：eea5527a450e
docker run -dit --name centos04 centos-hwj:7 bash  # 容器id: 6ca85a8dffc7
docker exec -it b22d43ffe7d9 ping centos03  # 发现不能ping通centos03


docker network connect mynet eea5527a450e  #将centos03加入mynet网络，实际上多了一个ip
docker network connect mynet 6ca85a8dffc7  
	# docker exec -it 6ca85a8dffc7 ip addr 发现6ca85a8dffc7有两个ip地址
docker exec -it b22d43ffe7d9 ping centos03  # 加入网络后就可以正常ping通centos03
```



# Docker-Compose

> 下载安装docker-compose

```shell
curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose 
chmod 777 /usr/local/bin/docker-compose
docker-compose version # 查看版本看是否下载成功
```

> hello-word demo 参考官网： https://docs.docker.com/compose/gettingstarted/

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

> Docker build 太慢解决方法

因为Alpine基础镜像用的是apk包管理工具，国内访问apk仓库极度缓慢，所以换成国内的仓库

参考https://yeasy.gitbook.io/docker_practice/os/alpine

```shell
# 原本是 RUN apk add --no-cache gcc musl-dev linux-headers
# 在apk前面加上 sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories &&
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories && apk add --no-cache gcc musl-dev linux-headers
```

pip源也换成国内的！

参考https://blog.csdn.net/qq_40244755/article/details/112388320

```shell
# RUN pip install -U pip
RUN pip config set global.index-url http://mirrors.aliyun.com/pypi/simple
RUN pip config set install.trusted-host mirrors.aliyun.com
```





## docker-compose常用命令

```shell
docker-compose build
docker-compose up
```



实战: 部署wordpress

参考https://docs.docker.com/samples/wordpress/ ， 如果链接失效就去搜索docker 文档里搜 wordpress





# Docker-Swarm

[工作机制](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)

![Swarm mode cluster](docker.assets/swarm-diagram.png)

## 常用命令

```shell
docker swarm init
  # demo: docker swarm init --advertise-addr 192.168.0.111
  # 会生成一个swarm集群
docker swarm join
  # demo: docker swarm join-token manager  # 会打印加入集群成为manager的命令
  # demo: docker swarm join-token worker # # 会打印加入集群成为worker的命令
docker node ls  # 查看节点
docker swarm leave # 离开集群
```



执行docker swarm init --advertise-addr 192.168.0.111会打印， **这个时候需要开放2377端口！**

```shell
Swarm initialized: current node (stw3wnedp6yurwp5lczwaulp7) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4qe8r3jrbwejpv7vlino8xcj8y6npebypm6z0l23dc0csqp6sd-8tq8hhga99yk8p0oq4njdxyks 192.168.0.111:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```



参考 [Docker swarm集群增加节点](https://www.cnblogs.com/zoujiaojiao/p/10886262.html)



## warm 网络

overlay

ingress, 就是特殊的overlay网络

这些内容有空再继续深入研究





# Docker服务



## 常用命令

```shell
docker service create  # 创建服务
  # demo: docker service create -p 8888:80 --name my-nginx nginx
docker service ls # 查看有哪些服务
docker service ps [服务名] # 查看服务
  # demo： docker service ps my-nginx
docker service update # 动态更新容器, 可以扩缩容
  # demo: docker service update --replicas 5 my-nginx
docker service scale # 
  # demo: docker service scale my-nginx=5
docker service rm # 移除服务
  # demo: docker service rm my-nginx
```



遇到一个问题：Pool overlaps with other one on this address space ，导致服务的5个容器都创建在同一台机器上。

解决办法：`docker network prune`

参考：

https://github.com/maxking/docker-mailman/issues/85

https://blog.csdn.net/x1097374154/article/details/105481211



# 其他



## docker stack

```shell
# 单机部署项目
docker-compose up -d wordpress.yaml
# 集群部署项目
docker stack deploy  # 百度搜一下， 有空再了解
```



## docker secret

安全相关， 配置密码，证书



## docker config

多个容器统一配置



##  UFS联合文件系统

对文件系统进行分层管理， 一次修改提交就是一层

bootfs 引导加载内核

rootfs 包含了基本命令，工具， 程序库等

容器比启动的比VM虚拟机快的原因： 直接使用宿主机的内核，  所以效率很快！



## 更换docker存储目录

1. 停止docker服务： systemctl stop docker.service
2. 默认工作路径是/var/lib/docker/， 将其mv或者cp到空间大的目录， 比如/data/docker
3. 修改 /etc/docker/daemon.json 的graph字段，指定为比如/data/docker
4. 启动docker服务： systemctl start docker.service



镜像配置目录： /etc/docker/daemon.json

默认工作路径： /var/lib/docker/  包含了以下目录

```
buildkit  containers  image  network  overlay2  plugins  runtimes  swarm  tmp  trust  volumes
```





