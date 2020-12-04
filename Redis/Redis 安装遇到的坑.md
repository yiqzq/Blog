### 安装 Docker Engine-Community

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：Redis 安装遇到的坑

使用的服务器：华为云

使用的是xhell远程连接服务器

## 坑

**坑一**

服务器没有gcc环境

`yum -y install gcc-c++`

**坑二**

没有tcl

`yum install tcl`

**坑三**

我的服务器使用最新版本6.0.3 有问题，make失败，换成低版本5.0.x 即成功

**安装过程**

```python3
wget http://download.redis.io/releases/redis-5.0.2.tar.gz
tar xzf redis-5.0.2.tar.gz
cd redis-5.0.2
make
cd src
make install PREFIX=/opt/redis
make test
```

## redis服务端的启动

进入/opt/redis/bin目录

使用命令`./redis-server`

看到如下页面，表示redis启动成功

<img src="https://i.loli.net/2020/05/19/fjIE7GcPaz4k8uh.png" alt="image-20200519114918578" style="zoom:150%;" />

## redis客户端的启动

进入/opt/redis/bin目录

使用命令`./redis-cli`

完整命令 `./redis-cli -h IP地址 -p 端口` 默认本机IP，端口6379

下图表示客户端启动成功

<img src="https://i.loli.net/2020/05/19/Ned63oM4xs5TjKY.png" alt="image-20200519115218605" style="zoom:150%;" />

## 简单使用演示

<img src="https://i.loli.net/2020/05/19/q9vOldybszg4uLZ.png" alt="image-20200519115821777" style="zoom: 150%;" />

## 使用配置文件

需要将下载文件中的redis.conf移动到安装目录中

在我这里，就是将/root/redis-5.0.2/redis.conf 中的移动到/opt/redis/bin/

将目录跳至redis-5.0.2，然后使用命令` cp redis.conf /opt/redis/bin/`

## 配置文件内容详情

以下只是列举了一部分

| 配置项                                                       | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `daemonize no`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ） |
| `port 6379`                                                  | 指定 Redis 监听端口，默认端口为 6379                         |
| `bind 127.0.0.1`                                             | 绑定的主机地址，若要使外界可以访问，注释即可                 |
| `timeout 300`                                                | 当客户端闲置多长秒后关闭连接，如果指定为 0 ，表示关闭该功能  |
| `loglevel notice`                                            | 指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice |
| `databases 16`                                               | 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id |
| `save  `Redis 默认配置文件中提供了三个条件：<br>**save 900 1**<br>**save 300 10**<br>**save 60 10000**<br>表示 900 秒（15 分钟）内有 1 个更改<br>300 秒（5 分钟）内有 10 个更改<br>以60 秒内有 10000 个更改。 | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合 |
| `dbfilename dump.rdb`                                        | 指定本地数据库文件名，默认值为 dump.rdb                      |
| `rdbcompression yes`                                         | 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大 |
| `dir ./`                                                     | 指定本地数据库存放目录                                       |
| `masterauth `                                                | 当 master 服务设置了密码保护时，slav 服务连接 master 的密码  |
| `requirepass foobared`                                       | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭 |
| ` maxclients 128`                                            | 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息 |
| `maxmemory `                                                 | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
| `appendfilename appendonly.aof`                              | 指定更新日志文件名，默认为 appendonly.aof                    |
| `vm-max-memory 0`                                            | 将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0 |
| `vm-page-size 32`                                            | Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值 |
| `vm-pages 134217728`                                         | 设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。 |
| `glueoutputbuf yes`                                          | 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启 |

指定配置文件启动redis，`./redis-server /opt/redis/bin/redis.conf `

## 远程连接redis

1. 注释掉配置文件中的`bind 127.0.0.1` 或者改成bind 0.0.0.0 ,这主要是为了允许外网访问

2. 修改requirepass ，即设置密码

3.  protected-mode 属性设置no

4. 如果开启了防火墙就关闭防火墙或者开放6379端口

   **一些防火墙有关的指令**

   启动防火墙：`systemctl start firewalld`

   禁用防火墙：`systemctl stop firewalld`

   设置开机启动：`systemctl enable firewalld`

   重启防火墙：`firewall-cmd --reload`

   查看状态：`systemctl status firewalld  或者 firewall-cmd --state`

   查看指定区域所有打开的端口：`firewall-cmd --zone=public --list-ports`
   在指定区域打开端口（记得重启防火墙）：`firewall-cmd --zone=public --add-port=80/tcp(永久生效再加上 --permanent)`

5. 可以使用Another.Redis.Desktop.Manager这款图形客户端软件

6. 有一个很坑的地方，参照网上的教程，使用ifconfig指令的ip地址死活远程连接不上，但是使用服务器的公网ip却连接上了，感觉很坑。

   ## Docker 安装redis

   1. 首先要安装docker

      **设置仓库**

      安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

**使用以下命令来设置稳定的仓库。**

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

 **安装 Docker Engine-Community**

安装最新版本的 Docker Engine-Community 和 containerd

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

启动 Docker。

```
$ sudo systemctl start docker
```

2. 查看docker版本

```
   docker version
```

3. 下载镜像

```
docker pull redis:5.0.2
```

4. 查看结果

```
docker image
```

5. 创建并运行容器

```
docker run -d --name redis-6379 -p 6379:6379 redis:5.0.2 --requirepass "123456"
```

6. 查看结果

```
docker ps
```

<img src="https://i.loli.net/2020/05/19/v3SdEnapsJ12M6Y.png" alt="image-20200519195220264" style="zoom:150%;" />

7. 进入容器

```
docker exec -it redis-6379 bash
```

