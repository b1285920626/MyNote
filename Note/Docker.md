# 1，安装

## 1.1卸载旧版本

较旧的 Docker 版本称为 docker 或 docker-engine 。如果已安装这些程序，请卸载它们以及相关的依赖项。

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



## 1.2安装 Docker Engine-Community

### 使用 Docker 仓库进行安装

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker。

**设置仓库**

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```shell
sudo yum install  -y  yum-utils 
                      device-mapper-persistent-data
                      lvm2
```



使用以下命令来设置稳定的仓库。

```shell
sudo yum-config-manager
     --add-repo 
     https://download.docker.com/linux/centos/docker-ce.repo
```

### 安装 Docker Engine-Community

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```



## 1.3 安装后报错

刚在新的Centos上安装Docker-CE,后运行

`docker run hello-world`

报错

`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`



**解决办法**:

```shell
systemctl daemon-reload
sudo service docker restart
sudo service docker status (should see active (running))
sudo docker run hello-world
```



## 1.4镜像加速

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

- Docker官方提供的中国镜像库：**https://registry.docker-cn.com**





对于使用 systemd 的系统(Ubuntu16.04+、Debian8+、CentOS7)，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：

```json
{
    "registry-mirrors":["https://registry.docker-cn.com"]
}
```



之后重新启动服务：

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



# 2,使用



## 2.1输出Hello world

```shell
docker run ubuntu:15.10 /bin/echo "Hello world"
```



各个参数解析：

- **docker:** Docker 的二进制执行文件。
- **run:** 与前面的 docker 组合来运行一个容器。
- **ubuntu:15.10** 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
- **/bin/echo "Hello world":** 在启动的容器里执行的命令

以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。



## 2.2运行交互式的容器

**下载**镜像，**https://hub.docker.com/** 查看官方镜像

```powershell
docker pull centos7
```

**删除镜像**

```shell
docker image rm [镜像名]

或

docker rmi [镜像名]
```

**查看**本地镜像列表

```powershell
docker image ls
```

**运行**

```shell
docker run -it ubuntu:15.10 /bin/bash
```

各个参数解析：

- **-t:** 在新容器内指定一个伪终端或终端。
- **-i:** 允许你对容器内的标准输入 (STDIN) 进行交互。

**启动已停止运行的容器**

查看所有的容器命令如下：

```shell
docker ps -a
```

[![img](C:\Users\FatWhite\Desktop\WorkSpace\03-笔记\Note\img\docker-container-psa.png)](https://www.runoob.com/wp-content/uploads/2016/05/docker-container-psa.png)

使用 docker start 启动一个已停止的容器：

```shell
docker start b750bbbcfd88 
```

**后台运行**

在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 **-d** 指定容器的运行模式。

```
$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```

**注：**加了 **-d** 参数默认不会进入容器，想要进入容器需要使用指令 **docker exec**（下面会介绍到）。

**停止一个容器**

停止容器的命令如下：

```
$ docker stop <容器 ID>
```

停止的容器可以通过 docker restart 重启：

```
$ docker restart <容器 ID>
```

**进入容器**

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

**attach 命令**

下面演示了使用 docker attach 命令。

```
$ docker attach 1e560fca3906 
```

[![img](C:\Users\FatWhite\Desktop\WorkSpace\03-笔记\Note\img\docker-attach.png)](https://www.runoob.com/wp-content/uploads/2016/05/docker-attach.png)

**注意：** 如果从这个容器退出，会导致容器的停止。

**exec 命令**

下面演示了使用 docker exec 命令。

```
docker exec -it 243c32535da7 /bin/bash
```

[![img](C:\Users\FatWhite\Desktop\WorkSpace\03-笔记\Note\img\docker-exec.png)](https://www.runoob.com/wp-content/uploads/2016/05/docker-exec.png)

**注意：** 如果从这个容器退出，不会导致容器的停止，这就是为什么推荐大家使用 **docker exec** 的原因。

更多参数说明请使用 **docker exec --help** 命令查看。



**退出**

我们可以通过运行 exit 命令或者使用 CTRL+D 来退出容器。



# 3，问题

## 3.1，官方mysql不启动

原因1：没有交换区（虚拟内存）...

**解决**

### 3.1.1给Linux创建交换区

1. 查看内存：

   ```shell
   free -m      //-m是显示单位为MB，-g单位GB
   ```

2. 创建一个文件：

   ```
   touch /etc/swapfile
   ```

3. 使用`dd`命令，来创建大小为2G的文件swapfile:

   ```
   dd if=/dev/zero of=/etc/swapfile bs=1M count=2048     //命令执行完需要等待一段时间
   ```

   `if`表示input_file输入文件
   `of`表示output_file输出文件
   `bs`表示block_size块大小
   `count`表示计数。
   这里，我采用了数据块大小为1M，数据块数目为2048，这样分配的空间就是2G大小。

   

4. 格式化交换文件：

   ```
   mkswap /etc/swapfile
   ```

5. 启用交换文件：

   ```
   swapon /etc/swapfile
   ```

如果要删除交换分区和交换文件，逆着上面的顺序操作:

1. 先删除/etc/fstab文件中添加的交换文件行

2. 停用交换文件

   ```
   swapoff  /etc/swapfile
   ```

3. 删除交换文件

   ```
   rm -fr  /etc/swapfile
   ```

   

   

### 3.1.2 增加交换区大小

1. 使用`free -m`查看在未增加swap之前虚拟内存的使用情况

2. 使用dd命令创建一个swap文件,大小为1G

   ```
   dd if=/dev/zero of=/etc/swapfile bs=1M count=2048  
   ```

   

3. 将文件格式转换为swap格式的

   ```
   mkswap  /home/swap
   ```

4. 再用swapon命令把这个文件分区挂载swap分区

   ```
   swapon  /home/swap
   ```




# 4，docker的build命令





# 5，docker-compose的使用











