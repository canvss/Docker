![Docker-Logo-700x394](imgs/Docker-Logo-700x394.png)

### docker的诞生

[docker](https://www.docker.com/)最初是由dotCloud公司创始人 [Solomon Hykes](https://github.com/shykes) 在法国期间发起的一个公司内部项目，最初底层利用了Linux容器技术（**LXC**）（在操作系统中实现资源隔离与限制）。从0.7版本以后基于[libcontainer](https://github.com/docker/libcontainer)开发。

- Hypervisor：一种运行在基础物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件。常见的VMware的workstation、ESXi、微软的Hyper-V或者思杰的XenServer。

- Container Runtime：通过Linux内核虚拟化能力管理多个容器，多个容器共享一套操作系统内核，因此摘掉了内核占用的空间及允许所需的耗时，使得容器极其轻量与快速。

- LXC  >  Libcontainer(创建一个隔离的，独立的namespace，也就是容器实例)

> 什么是容器技术：https://www.redhat.com/zh/topics/containers/whats-a-linux-container

### docker对比传统虚拟机

![2941939-20221219173421666-411559155](imgs/2941939-20221219173421666-411559155.png)

- 实现原理技术不同：
  - 虚拟机是用来进行硬件资源划分的完美解决方案，利用的是硬件虚拟化技术，如VT-x、AMD-V会通过hypervisor层来实现对资源的彻底隔离。
  - 容器是操作系统级别的虚拟化，利用的是内核的Cgroup和Namespace特性，通过软件来实现，仅仅是进程本身就可以实现互相隔离
- 使用资源方面：
  - Docker容器与主机共享操作系统内核，不同的容器之间可以共享部分系统资源，因此更加轻量级、消耗的资源更少。
  - 虚拟机会独占分配给自己的资源，不存在资源共享。各个虚拟机之间近乎完全隔离，更加重量级，也会消耗更多的资源。
- 应用场景不同：
  - 若需要资源的完全隔离并且不考虑资源的消耗，可以使用虚拟机。
  - 若隔离进程并且需要运用大量进程实例，应选择docker

| 特性       | 容器               | 虚拟机     |
| :--------- | :----------------- | :--------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱于       |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |
| 资源隔离   | 安全隔离           | 完全隔离   |

### Docker架构

- Docker使用C/S架构模式，可以远程API来管理和创建Docker容器。

- Docker容器通过Docker镜像来创建

- 容器与镜像的关系类似于面向对象的对象和类

#### 容器与镜像的关系

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

### Docker系统架构

![architecture (1)](imgs/architecture%20(1).svg)

- Docker Daemon：即Dockerd，Docker守护进程，监听Docker API请求并管理Docker对象。
- Image镜像：用于创建Docker容器的模版
- Container容器：容器是镜像运行的实体，一个镜像可以创建N个容器，每个处于运行的容器都包含着一个或多个相关的应用，且它们的运行不会干扰到其他容器；它们之间是互相隔离的。
- Repository仓库：用来保存相关一组镜像，这组镜像具有相同的镜像名称，都与镜像仓库名称相同。
- Tag标签：通过\<repository>:\<tag>即可唯一定位一个镜像。
- Registry镜像中心：存放着很多由官方、其他机构或个人创建的 Docker 仓库，Docker 用户可以直接从这些仓库中 pull 需要的镜像，也可以将自己制作的镜像 push 到 Docker 镜像 中心相应的仓库中。

### Docker引擎架构

- Docker Client：提供CLI工具，用于用户向Docker提交命令请求
- Dockerd：即Docker Daemon，功能有镜像构建、镜像管理、REST API、核心网络等，通过gRPC与Containerd通信
- Containerd：Container Daemon，是管理容器的生命周期，自身不会创建容器，调用Runc来完成容器的创建
- Runc：Run Container是OCI（开放容器倡议基金会）容器运行时规范的实现；Runc用于创建容器，本质是一个独立的容器运行时CKI工具。在fork出一个容器子进程后会启动该容器进程。在容器进程启动后，Runc会自动退出
- Shim：当Runc自动退出之前，会讲新容器进程的父进程指定为相应的Shim进程

### Docker隔离原理

- namespace（资源隔离）

| namespace | 系统调用参数  | 隔离内容                   |
| --------- | ------------- | -------------------------- |
| UTS       | CLONE_NEWUTS  | 主机和域名                 |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| PID       | CLONE_NEWPID  | 进程编号                   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等   |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）         |
| User      | CLONE_NEWUSER | 用户和用户组               |

- cgroups资源限制
  - 资源限制：限制任务使用的资源总额，并在超过这个配额时发出提示
  - 优先级分配：分配CPU时间片数量及磁盘IO宽带大小、控制任务运行的优先级
  - 资源统计：统计系统资源使用量，如CPU使用时长、内存用量等
  - 任务控制：对任务执行挂载、恢复等操作

| 子系统                          | 功能                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| cpu                             | 使用调度程序控制任务对CPU的使用                              |
| cpuacct(CPU Accounting)         | 自动生成cgroup中任务对CPU资源使用情况的报告                  |
| cpuset                          | 为cgroup中的任务分配独立的CPU（多处理器系统时）和内存        |
| devices                         | 开启或关闭cgroup中任务对设备的访问                           |
| freezer                         | 挂起火恢复cgroup中的任务                                     |
| memory                          | 设定cgroup中任务对内存使用量的限定，并生成这些任务对内存资源使用情况的报告 |
| perf_event(Linux CPU性能探测器) | 使cgroup中的任务可以进行统一的性能测试                       |
| net_cls(Docker未使用）          | 通过登记识别标记网络数据包，从而允许Linux流量监控程序（Traffic Controller）识别从具体cgroup中生成的数据包 |

### Docker仓库

- 公有 Docker Registry：公有服务是开放给用户使用、允许用户管理镜像的Registry服务。
  - [Docker Hub](https://hub.docker.com/)
  - [CoreOS](https://coreos.com/)
  - [Quay.io](https://quay.io/repository/)
- 私有 Docker Registry：用户可以本地搭建私有Docker Registry；Docker官方提供了Docker Registry镜像了，可以直接作为私有Registry服务
  - [Docker Trusted Registry](https://docs.docker.com/)
  - [VMWare Harbor](https://github.com/vmware/harbor)
  - [Sonatype Nexus](https://www.sonatype.com/docker) 

### Docker安装

>官方安装教程：https://docs.docker.com/engine/install/centos/#install-using-the-repository

**1.安装必要的工具：**

```shell
[root@docker ~]# yum install -y yum-utils
```

**2.添加软件源信息：**

```shell
[root@docker ~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**3.安装docker引擎**

```shell
[root@docker ~]# yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```shell
#启动docker
[root@docker ~]# systemctl start docker
#查看docker版本信息
[root@docker ~]#  docker version
Client: Docker Engine - Community
 Version:           23.0.6
 API version:       1.42
 Go version:        go1.19.9
 Git commit:        ef23cbc
 Built:             Fri May  5 21:19:08 2023
 OS/Arch:           linux/amd64
 Context:           default
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

**4.测试hello-world**

```shell
#运行hello-world
[root@docker ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete 
Digest: sha256:9eabfcf6034695c4f6208296be9090b0a3487e20fb6a5cb056525242621cf73d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**5.查看docker镜像**

```
#查看docker镜像
[root@docker ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    9c7a54a9a43c   4 days ago   13.3kB
```

**6.配置阿里云镜像**

```shell
#配置docker阿里云镜像
[root@docker ~]# mkdir -p /etc/docker/
[root@docker ~]# tee /etc/docker/daemon.json <<-'EOF'
> {
>  "registry-mirrors": ["https://anurpwmt.mirror.aliyuncs.com"]
>	} 
EOF
{
 "registry-mirrors": ["https://anurpwmt.mirror.aliyuncs.com"]
}
[root@docker ~]# systemctl daemon-reload
[root@docker ~]# systemctl restart docker
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.
```

### Docker常用命令

![](imgs/cmd_logic.png)

| 命令      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| attach    | 绑定到运行中容器的标准输入、输出，以及错误流（这样似乎也能进入容器内容，但是一定小心，它们操作的就是控制台，控制台的退出命令会生效，比如redis、nginx） |
| build     | 从一个 Dockerfile 文件构建镜像                               |
| commit    | 把容器的改变 提交创建一个新的镜像                            |
| cp        | 容器和本地文件系统间 复制 文件/文件夹                        |
| create    | 创建新容器，但并不启动(注意与docker run 的区分)需要手动启动。start\stop |
| diff      | 检查容器里文件系统结构的更改【A:添加文件或目录 D:文件或者目录删除 C:文 件或者目录更改】 |
| events    | 获取服务器的实时事件                                         |
| exec      | 在运行时的容器内运行命令                                     |
| export    | 导出 **容器** 的文件系统为一个tar文件。commit是直接提交成镜像，export是导出成文 件方便传输 |
| history   | 显示镜像的历史                                               |
| images    | 列出所有镜像                                                 |
| import    | 导入tar的内容创建一个镜像，再导入进来的镜像直接启动不了容器。<br/> /docker-entrypoint.sh nginx -g 'daemon o;'<br/> docker ps --no-trunc 看下之前的完整启动命令再用他 |
| info      | 显示系统信息                                                 |
| inspect   | 获取docker对象的底层信息                                     |
| kill      | 杀死一个或者多个容器                                         |
| load      | 从 tar 文件加载镜像                                          |
| login     | 登录Docker registry                                          |
| logout    | 退出Docker registry                                          |
| logs      | 获取容器日志;容器以前在前台控制台能输出的所有内容，都可以看到 |
| pause     | 暂停一个或者多个容器                                         |
| port      | 列出容器的端口映射                                           |
| ps        | 列出所有容器                                                 |
| pull      | 从registry下载一个image 或者repository                       |
| push      | 给registry推送一个image或者repository                        |
| rename    | 重命名一个容器                                               |
| restart   | 重启一个或者多个容器                                         |
| rm        | 移除一个或者多个容器                                         |
| rmi       | 移除一个或者多个镜像                                         |
| run       | 创建并启动容器                                               |
| save      | 把一个或者多个 **镜像** 保存为tar文件                        |
| search    | 去docker hub寻找镜像                                         |
| start     | 启动一个或者多个容器                                         |
| stats     | 显示容器资源的实时使用状态                                   |
| stop      | 停止一个或者多个容器                                         |
| tag       | 给源镜像创建一个新的标签，变成新的镜像                       |
| top       | 显示正在运行容器的进程                                       |
| unpause   | pause的反操作                                                |
| update    | 更新一个或者多个docker容器配置                               |
| container | 管理容器                                                     |
| network   | 管理网络                                                     |
| volume    | 管理卷                                                       |

### Docker镜像 

#### 查找镜像

```shell
[root@docker ~]# docker search --help
Usage:  docker search [OPTIONS] TERM
Search Docker Hub for images
Options:
	#根据提供条件过滤输出
  -f, --filter filter   Filter output based on conditions provided
  #用Go模版打印出指定的搜索结果
  --format string   Pretty-print search using a Go template
 	#搜索结果的最大数量（默认值为25）
 	--limit int       Max number of search results
 	#不要截断输出
  --no-trunc        Don't truncate output
```

```shell
[root@docker ~]# docker search tomcat
```

#### 获取镜像

- docker pull [OPTIONS] NAME[:TAG|@DIGEST]

```shell
#不指定版本号，代表laster版本
[root@docker ~]# docker pull tomcat 
```

#### 列出镜像

```shell
[root@docker ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    9c7a54a9a43c   6 days ago   13.3kB
redis         latest    116cad43b6af   7 days ago   117MB
[root@docker ~]# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    9c7a54a9a43c   6 days ago   13.3kB
redis         latest    116cad43b6af   7 days ago   117MB
```

#### 虚悬镜像(Dangling Image)

- 镜像列表中，有一种特殊的镜像，这个镜像既没有仓库名，也没有标签

```shell
#删除虚悬镜像
[root@docker ~]# docker image prune
```

> 这个镜像原本是有镜像名和标签的，原来为 tomcat:8.0，随着官方镜像维护，发布了新版本后，重新 docker pull tomcat:8.0 时，tomcat:8.0 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消， 从而成为了 。除了 docker pull 可能导致这种情况，docker build 也同样可以导致这种现象。由于新旧镜像同 名，旧镜像名称被取消，从而出现仓库名、标签均为 的镜像。这类无标签镜像也被称为 虚悬镜像(dangling image) 。 一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的.

#### 删除本地镜像

- docker rmi [OPTIONS] IMAGE [IMAGE...]

```shell
[root@docker docker]# docker image rmi hello-world:latest 
Untagged: hello-world:latest
Untagged: hello-world@sha256:9eabfcf6034695c4f6208296be9090b0a3487e20fb6a5cb056525242621cf73d
Deleted: sha256:9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d
Deleted: sha256:01bb4fce3eb1b56b05adf99504dafd31907a5aadac736e36b27595c8b92f07f1

[root@docker docker]# docker images 
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
redis        latest    116cad43b6af   7 days ago   117MB
```

#### 查看本地镜像的IMAGE ID

```shell
[root@docker docker]# docker images -q
116cad43b6af
```

#### 查看镜像的制作历史

```shell
[root@docker docker]# docker history redis
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
116cad43b6af   7 days ago   /bin/sh -c #(nop)  CMD ["redis-server"]         0B        
<missing>      7 days ago   /bin/sh -c #(nop)  EXPOSE 6379                  0B        
<missing>      7 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      7 days ago   /bin/sh -c #(nop) COPY file:e873a0e3c13001b5…   661B      
```

#### 保存镜像

- save选项将本地仓库的镜像保存当前目录下

```shell
[root@docker docker]# docker save -o redis.tar redis
[root@docker docker]# ll redis.tar 
-rw------- 1 root root 120723968 May 11 15:37 redis.tar
```

- load -i 选项将打包的镜像文件导入到本地Docker仓库

```shell
[root@docker docker]# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
[root@docker docker]# docker load -i redis.tar 
8553b91047da: Loading layer [==================================================>]  84.01MB/84.01MB
a29f3c086730: Loading layer [==================================================>]  338.4kB/338.4kB
bee68ae43a83: Loading layer [==================================================>]  4.229MB/4.229MB
df132c87bdb2: Loading layer [==================================================>]  32.11MB/32.11MB
c4afa995e3ec: Loading layer [==================================================>]  2.048kB/2.048kB
47998e638469: Loading layer [==================================================>]  4.096kB/4.096kB
Loaded image: redis:latest
[root@docker docker]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
redis        latest    116cad43b6af   7 days ago   117MB
```

### Docker容器

#### 启动容器

**docker run 参数 镜像名称:tag 执行命令**

**docker create [OPTIONS] IMAGE [COMMAND] [ARG...]** 

**常用参数**

- -i 保持和docker容器内的交互，启动容器时，运行的命令结束后，容器依然存活，没有退出（默认是会退出，即停止）
- -t 为容器的标准输入虚拟的一个tty
- -d 后台运行容器
- --rm 容器在启动后，执行完成命令或程序后就销毁
- --name 给容器起一个名称
- -p 宿主机端口:内部端口

```shell
#创建好容器不会启动，需要手动启动
[root@docker ~]# docker create redis
3c4c54c04381b207435ffb6c39e0997fcce0e4b992204eabd79a861349502e76
[root@docker ~]# docker ps -a|grep redis
3c4c54c04381   redis     "docker-entrypoint.s…"   16 seconds ago   Created                                                            tender_kare

[root@docker docker]# docker run -d --name redis1 -p 6379:6379 redis
6e819bf81a479f9667871c8cf1be6e302e7e54221c1aacfa55e917bead2c2533
```

#### 查看容器状态

- docker ps  #查看运行的容器
- docker ps -a   #查看所有容器（包含运行和停止）
- docker container ls
- docker container ls -a 

```shell
[root@docker docker]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                       NAMES
6e819bf81a47   redis     "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   redis1

[root@docker docker]# docker container ls -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
6e819bf81a47   redis     "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   redis1
```

#### 停止容器

- docker stop 容器名

- docker container stop 容器名

```shell
[root@docker docker]# docker stop 6e819bf81a47
6e819bf81a47
[root@docker docker]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                      PORTS     NAMES
6e819bf81a47   redis     "docker-entrypoint.s…"   5 minutes ago   Exited (0) 40 seconds ago             redis1

[root@docker ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED             STATUS             PORTS                                       NAMES
9ac93190c96d   tomcat    "catalina.sh run"   About an hour ago   Up About an hour   0.0.0.0:8081->8080/tcp, :::8081->8080/tcp   tomcat-8081
f819a6c49c20   tomcat    "catalina.sh run"   About an hour ago   Up About an hour   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   tomcat-8080

#停止所有容器
[root@docker ~]# docker stop $(docker ps -q)
9ac93190c96d
f819a6c49c20
```

#### 启动已停止容器

- docker start 容器id

```shell
[root@docker docker]# docker start 6e819bf81a47
6e819bf81a47

[root@docker docker]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
6e819bf81a47   redis     "docker-entrypoint.s…"   6 minutes ago   Up 6 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   redis1
```

#### 重启容器

- docker restart 容器id

```shell
[root@docker docker]# docker restart 6e819bf81a47
6e819bf81a47
```

#### 删除容器

- 删除容器需要停止容器
- docker rm 容器id

```shell
[root@docker docker]# docker stop 6e819bf81a47
6e819bf81a47
[root@docker docker]# docker rm 6e819bf81a47
6e819bf81a47
[root@docker docker]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

[root@docker ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS                        PORTS     NAMES
9ac93190c96d   tomcat    "catalina.sh run"        About an hour ago   Exited (143) 47 seconds ago             tomcat-8081
f819a6c49c20   tomcat    "catalina.sh run"        About an hour ago   Exited (143) 47 seconds ago             tomcat-8080
0df765bf4c4f   redis     "docker-entrypoint.s…"   2 hours ago         Exited (0) 2 hours ago                  quirky_nobel

#删除所有容器
[root@docker ~]# docker rm $(docker ps -aq)
9ac93190c96d
f819a6c49c20
0df765bf4c4f
```

####  查看后台运行日志

- docker logs 容器id/容器名

```shell
[root@docker docker]# docker logs 0df765bf4c4f
1:C 11 May 2023 08:06:21.579 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 11 May 2023 08:06:21.579 # Redis version=7.0.11, bits=64, commit=00000000, modified=0, pid=1, just started

#查看tomcat末尾5行日志
[root@docker ~]# docker logs -f --tail=5 tomcat-8081
11-May-2023 09:08:26.131 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/10.0.14]
11-May-2023 09:08:26.146 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
11-May-2023 09:08:26.177 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [145] milliseconds
11-May-2023 09:47:56.426 INFO [Catalina-utility-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/tomcat/webapps/ROOT]
11-May-2023 09:47:56.771 INFO [Catalina-utility-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/tomcat/webapps/ROOT] has finished in [343] ms
```

#### Docker进入容器

- 命令：docker exec -it 容器id/容器名 bash

```shell
[root@docker /]# docker exec -it tomcat-8080 bash
#退出tomcat容器bash解释器
root@f819a6c49c20:/usr/local/tomcat/webapps# echo tomcat-8080 >> ROOT/index.html
root@9ac93190c96d:/usr/local/tomcat# exit
exit
```

> docker pull tomcat是最小安装，没有tomcat静态资源
>
> 默认容器内linux包是最小安装。只提供最基本的命令；exit不会导致容器的停止。

#### 宿主机和容器之间交换文件

- 容器复制到宿主机：docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
- 宿主机复制到容器：docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

```shell
#将宿主机文件复制到容器中
[root@docker ~]# docker cp Docker-Logo-700x394.png tomcat-8080:/usr/local/tomcat/webapps/ROOT/
Successfully copied 5.12kB to tomcat-8080:/usr/local/tomcat/webapps/ROOT/

root@f819a6c49c20:/usr/local/tomcat/webapps/ROOT# ls
Docker-Logo-700x394.png  index.html
#将容器文件复制到宿主机中
[root@docker ~]# docker cp tomcat-8080:/usr/local/tomcat/webapps/ROOT/index.html /root/
Successfully copied 2.05kB to /root/
[root@docker ~]# ls
Docker-Logo-700x394.png  index.html
```

### 推送镜像

```shell
#运行中的容器保存到新的镜像
[root@docker ~]# docker commit -a canvs -m "v1" 630ac564b05c canvs/nginx:v1
sha256:b91253c34bbf8b1049a238a5dc9a677655105b067d5676608c467e206bace15b
[root@docker ~]# docker images | grep canvs/nginx
canvs/nginx   v1        b91253c34bbf   38 seconds ago   141MB
```

```shell
#登录docker hub
[root@docker ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: canvs
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
#推送到docker hub
[root@docker ~]# docker push canvs/nginx:v1
```

### Docker挂载

- Volumes（卷）：存储在主机文件系统的一部分中，该文件系统由Docker管理（在Linux上时"/var/lib/docker/volumes/"）。非Docker进程不应该修改文件系统这一部分。
  - 匿名卷：docker run -d -P -v :/etc/nginx nginx
  - 具名卷：docker run -d -P -v nginx:/etc/nginx nginx
- Bind mounts（绑定挂载）：可以在任何地方存储在主机系统上。它们甚至可能是重要的文件或目录。Docker主机或Docker容器上的非Docker进程可以随时对其修改。
  - docker run -d -P -v /root/nginx:/etc/nginx nginx
- tmpfs mounts（临时挂载）：仅存储在主机系统中的内存中，并且永远不会写入主机的文件系统。

![553377-20211031122438814-2046080875](imgs/553377-20211031122438814-2046080875.png)

#### 数据卷特性

- 数据卷可以在容器之间共享和重复利用数据
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

>通过镜像创建一个容器。容器一旦销毁，则容器内的数据将一并被删除；容器中的数据不是持久化状态的；数据卷的目的就是数据的持久化，完全独立于容器的生命周期，因此docker不会再容器删除时删除其挂载的数据卷。

### 数据卷的应用

**创建数据卷**

```shell
[root@docker ~]# docker volume create web_volume
web_volume
#创建数据卷之后，默认会存放到目录: /var/lib/docker/volume/数据卷名称/_data目录下
[root@docker ~]# ll  /var/lib/docker/volumes/web_volume/_data/
total 0
```

**查看数据卷**

```shell
[root@docker ~]# docker volume inspect web_volume
[
    {
        "CreatedAt": "2023-05-11T18:10:50+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/web_volume/_data",
        "Name": "web_volume",
        "Options": null,
        "Scope": "local"
    }
]
#查看所有数据卷
[root@docker ~]# docker volume ls
DRIVER    VOLUME NAME
local     74e526402999a3fc7255ebb3b2f4a35e2a7165c5b47221df1180c45d7c1560d0
local     77d01da6df72f994f20a6b5759c8ade814abe432e32ffbdf847d4591896270ab
local     c83e3fbac31c1ff25d2076c5bb48a32b7ce0d13feb923e06699206754d49c349
local     dcb47d4e9a06a64384391c4a02a748837ac4e936c978e05cbac63b1d01c1e199
local     ea9fe6727a6020fc6b1646d525195afbf63f29088f9c52be056acaf3bda9ce5d
local     web_volume
```

**应用数据卷**

```shell
[root@docker ~]# docker run -d -it --name webtomcat -p 8080:8080 -v web_volume:/usr/local/tomcat/webapps/ROOT/ tomcat
4cc619d7a761057e84660830689764ace81409fd1f905a67ea99d26d3f9472a0

#在数据卷中添加文件
[root@docker _data]# ls
website

#容器目录下会自动同步已挂载数据卷中的内容
root@4cc619d7a761:/usr/local/tomcat/webapps/ROOT# ls
website
```

**删除数据卷**

```shell
[root@docker ~]# docker volume rm web_volume 
web_volume
#删除无用的数据卷
[root@docker ~]# docker volume prune
```

### Nginx

```shell
#安装Nginx
[root@docker ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

#运行nginx容器
[root@docker ~]# docker run -it --name nginx-80 --rm -d -p 80:80 nginx
ba0036f27aa9eeafb1baff2faa57b75ee5d4747b664d83b50726a5d76224c524

#查看nginx容器
[root@docker ~]# docker ps |grep nginx
ba0036f27aa9   nginx     "/docker-entrypoint.…"   3 minutes ago       Up 2 minutes    0.0.0.0:80->80/tcp, :::80->80/tcp           nginx-80

#进入nginx容器
[root@docker ~]# docker exec -it nginx-80 bash
root@ba0036f27aa9:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx

#将nginx容器中的配置文件nginx.conf conf.d复制到宿主机/usr/local/docker/nginx/conf/下
[root@docker ~]# mkdir -p /usr/local/docker/nginx
[root@docker ~]# docker cp nginx-80:/etc/nginx/nginx.conf /usr/local/docker/nginx/conf
Successfully copied 2.56kB to /usr/local/docker/nginx/
[root@docker ~]# docker cp nginx-80:/etc/nginx/conf.d /usr/local/docker/nginx/conf
Successfully copied 3.58kB to /usr/local/docker/nginx/

#停止nginx
[root@docker nginx]# docker stop nginx-80

[root@docker conf]# docker run --name nginx-80 -d -p 80:80 -v /usr/local/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/docker/nginx/conf/conf.d:/etc/nginx/conf.d -v /usr/local/docker/nginx/html:/usr/share/nginx/html -v /usr/local/docker/nginx/logs:/var/log/nginx nginx
630ac564b05cc1897d34a9e7d45ac76379f18a29e2bd4e0bfc97806228b547fc

[root@docker conf]# docker ps |grep nginx
630ac564b05c   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, :::80->80/tcp           nginx-80
```

#### 反向代理

- 修改conf.d/default.conf

```shell
server_name  exam_qf;
location / {
       proxy_pass http://127.0.0.1:8080;
 }
```

#### 负载均衡

- 修改nginx.conf

```shell
http{
	#配置负载均衡
  upstream nginxCluster{
          server 114.67.239.109:8080;
          server 114.67.239.109:8081;
          server 114.67.239.109:8082;
  }
}
```

- 修改conf.d/default.conf

```shell
location /{
     proxy_pass http://nginxCluster;
}
#location / {
#   root   /usr/share/nginx/html;
#    index  index.html index.htm;
# }
```

### Mysql

```shell
[root@docker /]# docker pull mysql
#启动；-e MYSQL_ROOT_PASSWORD='canvs'指定root密码
[root@docker /]# docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD='canvs' mysql
[root@docker /]# docker exec -it mysql bash
root@c771a20980d0:/# mysql -u root -p 
```

### 存储原理

```shell
[root@docker ~]# docker image inspect nginx
 "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b0a8808e8f762f08d831eb21fcdaf90de0af64651811d77c5aeb356c3de55a79/diff:/var/lib/docker/overlay2/fd7d4d68ae4949b758cddac41222d14ae7e202e2e421c294da3a14ffbd163f54/diff:/var/lib/docker/overlay2/c184ae272cbcf94f4b092923ffc74841aa1eb6771ad50525b4bb58d83ca9c3c5/diff:/var/lib/docker/overlay2/cff17f5604b4a16633e52232c98a159582e0f270eaa7f3276ff7b5fe423ccd61/diff:/var/lib/docker/overlay2/c519d5971ca202fcd066eee15991047fda5f9ee446accc9c96222eaea41a167a/diff",
                "MergedDir": "/var/lib/docker/overlay2/c124a30ca4b869946cea745cc1d57365622c2f096a3bcab63f01c98b80df019d/merged",
                "UpperDir": "/var/lib/docker/overlay2/c124a30ca4b869946cea745cc1d57365622c2f096a3bcab63f01c98b80df019d/diff",
                "WorkDir": "/var/lib/docker/overlay2/c124a30ca4b869946cea745cc1d57365622c2f096a3bcab63f01c98b80df019d/work"
            },
```

- LowerDir：底层目录；diff（只是存不同）；包含小型Linux和装好的软件

```shell
#小Linux系统
[root@docker diff]# ls /var/lib/docker/overlay2/c519d5971ca202fcd066eee15991047fda5f9ee446accc9c96222eaea41a167a/diff
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

#nginx配置文件
[root@docker diff]# ls /var/lib/docker/overlay2/cff17f5604b4a16633e52232c98a159582e0f270eaa7f3276ff7b5fe423ccd61/diff
docker-entrypoint.d  etc  lib  tmp  usr  var

#nginx启动命令
[root@docker diff]# ls /var/lib/docker/overlay2/c184ae272cbcf94f4b092923ffc74841aa1eb6771ad50525b4bb58d83ca9c3c5/diff
docker-entrypoint.sh

#
[root@docker diff]# ls /var/lib/docker/overlay2/fd7d4d68ae4949b758cddac41222d14ae7e202e2e421c294da3a14ffbd163f54/diff
docker-entrypoint.d
[root@docker diff]# ls /var/lib/docker/overlay2/b0a8808e8f762f08d831eb21fcdaf90de0af64651811d77c5aeb356c3de55a79/diff
docker-entrypoint.d
```

- MergedDir：合并目录；容器最终的 完整工作目录全内容都在合并目录；数据卷在容器层产生；所有的增删改查都在容器层
- UpperDir：上层目录
- WorkDir：工作目录（零时层），pid；

### 可视化界面-[Portainer](https://documentation.portainer.io/)

Portainer社区版2.0拥有超过50万的普通用户，是功能强大的开源工具集，可让您轻松地在Docker， Swarm，Kubernetes和Azure ACI中构建和管理容器。 Portainer的工作原理是在易于使用的GUI后面隐藏 使管理容器变得困难的复杂性。通过消除用户使用CLI，编写YAML或理解清单的需求，Portainer使部署 应用程序和解决问题变得如此简单，任何人都可以做到。 Portainer开发团队在这里为您的Docker之旅提 供帮助;

```shell
# 服务端部署
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
# 访问 9000 端口即可
#agent端部署
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent
```

### Dockerfile

| 指令        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| FROM        | 指定基础镜像                                                 |
| MAINTAINER  | 指定维护者信息，已经过时，可以使用LABEL maintainer=xxx 来替代 |
| RUN         | 运行命令 v                                                   |
| CMD         | 指定启动容器时默认的命令 v                                   |
| ENTRYPOINT  | 指定镜像的默认入口.运行命令 v                                |
| EXPOSE      | 声明镜像内服务监听的端口 v                                   |
| ENV         | 指定环境变量，可以在docker run的时候使用-e改变 v;会被固化到image的config里面 |
| ADD         | 复制指定的src路径下的内容到容器中的dest路径下，src可以为url会自动下载， 可以为tar文件，会自动解压 |
| COPY        | 复制本地主机的src路径下的内容到镜像中的dest路径下，但不会自动解压等 |
| LABEL       | 指定生成镜像的元数据标签信息                                 |
| VOLUME      | 创建数据卷挂载点                                             |
| USER        | 指定运行容器时的用户名或UID                                  |
| WORKDIR     | 配置工作目录，为后续的RUN、CMD、ENTRYPOINT指令配置工作目录   |
| ARG         | 指定镜像内使用的参数(如版本号信息等)，可以在build的时候，使用--build- args改变 v |
| OBBUILD     | 配置当创建的镜像作为其他镜像的基础镜像是，所指定的创建操作指令 |
| STOPSIGNAL  | 容器退出的信号值                                             |
| HEALTHCHECK | 健康检查                                                     |
| SHELL       | 指定使用shell时的默认shell类型                               |
