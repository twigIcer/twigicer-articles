# 虚拟机中用Docker安装并启动了Elasticsearch，浏览器却访问不了

> 注意：本文原发于CSDN：http://t.csdnimg.cn/MFvOn，现已被杭州城市开发者社区收录，转载请标明出处。

学习springCloud的过程及其痛苦，一直跟着黑马的视频走，但是随着技术栈的更新，许多技术的配置与黑马视频讲的会有出入，然后就会遇到一些很头疼的问题，有时候一个问题需要找很久的原因与解决方法，所以记录一下这些问题防忘吧(由于是尝试过程中解决了问题，没有截图，但说的还算详细)。

### 问题：

在学习到Elasticsearch时，我用docker安装并启动了Elasticsearch,前期过程挺顺利的，但是在用浏览器访问的时候出问题了，怎么也访问不到，提示拒绝访问。

### 解决过程：

#### 1. 防火墙问题：

网上大部分说的就是防火墙的问题，但是在刚学docker时，我就关闭了防火墙并且禁止了开机启动，给有需要的提供下命令吧：

```shell
firewall-cmd --state # 查看防火墙状态
systemctl stop firewalld.service # 停止firewall
systemctl disable firewalld.service # 禁止firewall开机启动
reboot # 重启虚拟机
```

#### 2. max_map_count太小：

第二种比较多的说法是：max_map_count太小了，但是我修改了之后问题依然没有解决，命令如下：

先查看max_map_count值（一般是65530，但如果是262144就不用改）：

```shell
cat /proc/sys/vm/max_map_count
65530
```

修改65530为262144:

```shell
#临时修改
sysctl -w vm.max_map_count=262144
 
#永久修改
vm.max_map_count=262144​
```

#### 3. 虚拟机内存不足以给ES分配：

还有说法是ES占用的内存比较多，如果虚拟机内存不足以分配给ES时会导致启动失败，解决方法：

```shell
#查看ES的容器id：
docker ps -a
 
#删除ES容器：
docker rm + 容器id
 
#新建ES容器（重点加上-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"）：
docker run -d \
--name es \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
-v es-plugins:/usr/share/elasticsearch/plugins \
-v /path/to/data/dir:/usr/share/elasticsearch/data \
--network es-net \
--privileged \
-p 9200:9200 \
-p 9300:9300 \
elasticsearch:7.12.1
```

这个是黑马视频中说到过的，所以我也是加上的，对我的问题没有帮助。

### 正解：挂载点目录问题：

#### 1. 查看日志：

我在寻求方法时，偶然发现，这个命令可以查看Elasticsearch的日志：

```shell
# 查看最新日志（默认情况下使用-f选项）
docker logs -f +容器id或者镜像名
 
# 查看特定时间段内的日志：
docker logs --since 2022-01-01 +容器id或者镜像名
 
# 仅查看错误日志：
docker logs --since 1d --grep ERROR +容器id或者镜像名
```

然后我查看了我的日志，发现在我浏览器访问ip:9200时，会出现这个错误并且此时我的容器会被自动删除：

```shell
ElasticsearchException[failed to bind service]; nested: FileSystemException[/usr/share/elasticsearch/data/nodes/0: Not a directory];
Likely root cause: java.nio.file.FileSystemException: /usr/share/elasticsearch/data/nodes/0: Not a directory
```

#### 2. 尝试修复：

大致意思就是说我的挂载目录不存在，但是我单独创建了目录后，还是会报错：

```shell
uncaught exception in thread [main]
ElasticsearchException[failed to bind service]; nested: AccessDeniedException[/usr/share/elasticsearch/data/nodes];
Likely root cause: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/data/nodes
```

然后我就针对目录做了一系列的我能想到的方法，但是都没解决问题。

#### 3. 正确方法：

无奈之下，我只能完全删除了Elasticsearch的镜像以及容器，重新安装，但是在安装之前先创建挂载点文件目录！

下面为docker安装lasticsearch的完整步骤：

(1) 创建一个网络，方便后期部署kibana:

```shell
docker network create es-net
```

(2) docker拉取Elasticsearch，不知道为什么在拉取Elasticsearch时，必须加tag，不能直接用latest:

```shell
# 必须选取一个tag,以7.12.1为例：
docker pull elasticsearch:7.12.1
```

(3) 创建搭载目录（重点！！！很多教程都没有）:

```shell
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data
 
# 将http.host: 0.0.0.0写入到es配置文件中，代表能被远程的任何机器访问：
echo "http.host: 0.0.0.0" > /mydata/elasticsearch/config/elasticsearch.yml
```

(4) 创建容器：

```shell
docker run -d \
--name es \
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
-v es-plugins:/usr/share/elasticsearch/plugins \
-v /path/to/data/dir:/usr/share/elasticsearch/data \
--network es-net \
--privileged \
-p 9200:9200 \
-p 9300:9300 \
elasticsearch:7.12.1
```

> 解释：
>
> -e "cluster.name=es-docker-cluster"：设置集群名称
>
> -e "http.host=0.0.0.0"：监听的地址，可以外网访问
>
> -e "ES_JAVA_OPTS=-Xms512m -Xmx512m"：内存大小
>
> -e "discovery.type=single-node"：非集群模式
>
> -v es-data:/usr/share/elasticsearch/data：挂载逻辑卷，绑定es的数据目录
>
> -v es-plugins:/usr/share/elasticsearch/plugins：挂载逻辑卷，绑定es的插件目录
>
> --privileged：授予逻辑卷访问权
>
> --network es-net ：加入一个名为es-net的网络中
>
> -p 9200:9200：端口映射配置

(5) 浏览器访问：虚拟机ip:9200,出现以下页面代表访问成功(edge浏览器出现的为json格式)：

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/d87df99b92e74d908a33565c08b0228a.png)

 问题解决！！！

### 总结：

这次的问题主要是，事前没有准备挂载点目录，其实黑马的视频和很多教程也没有这一步，所以绕了很大的弯子，花费很长时间的另一个原因是：我没有及时的查看日志，其实程序员遇到错误时第一项该做的就是查看日志，而我首先是去盲目的搜索答案了，也算让我长长记性吧。