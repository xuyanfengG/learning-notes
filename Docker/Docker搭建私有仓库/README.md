## 一、安装Docker官方仓库镜像：Registry

#### 1.拉取镜像

```shell
docker pull registry
```

#### 2.启动

```shell
docker run -d -p 5000:5000 -v /data0/registry:/var/lib/registry --restart=always --name registry registry:latest 
```

参数说明：

* -v：把宿主机的/data/registry目录绑定到容器/var/lib/registry目录(这个目录是registry容器中存放镜像文件的目录)，来实现数据的持久化； 
* -p：映射端口；访问宿主机的5000端口就访问到registry容器的服务了； 
* --restart=always：这是重启的策略，假如这个容器异常退出会自动重启容器； 
* --name registry：创建容器命名为registry，你可以随便命名；
*  registry:latest：这个是刚才pull下来的镜像；

#### 3.浏览器访问：http://localhost:5000/v2/_catalog

```json
{"repositories":[]}
```

#### 4.修改容器配置文件

（1）获取容器ID

```shell
docker ps
```

（2）修改配置文件

```shell
docker exec -it <CONTAINER ID> vi /etc/docker/registry/config.yml
```

（3）添加相关配置

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

（4）重启容器

#### 5.修改系统配置文件

```shell
vim /etc/docker/daemon.json
```

添加以下配置

```json
{"insecure-registries": [ "[IP]:5000"]}
```

```shell
vim /etc/sysconfig/docker
```

添加以下配置

```shell
# centos6 
other_args="--insecure-registry [IP]:5000" 
# centos7 
OPTIONS="--insecure-registry [IP]:5000"
```

#### 6.重启容器并测试

（1）下载测试镜像

```
docker pull busybox
```

（2）标记该测试镜像需要推送到私有仓库

```
docker tag  busybox:latest  [IP]:5000/busybox:20210526
```

（3）将镜像推送至私有仓库

```shell
docker push [IP]:5000/busybox:20210526
```

（4）查看仓库信息

```shell
curl http://10.0.1.200:5000/v2/_catalog|python -m json.tool
```

（5）拉取镜像

```shell
docker pull [IP]:5000/busybox:20210526
```

## 二、 安装第三方镜像仓库：Harbor

> docker 官方提供的私有仓库 registry，用起来虽然简单 ，但在管理的功能上存在不足。 Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，harbor使用的是官方的docker registry(v2命名是distribution)服务去完成。harbor在docker distribution的基础上增加了一些安全、访问控制、管理的功能以满足企业对于镜像仓库的需求。

#### 1.下载harbor

https://github.com/goharbor/harbor/releases

https://storage.googleapis.com/harbor-releases/

#### 2.解压并修改配置文件

```shell
#hostname 改为本地ip，非 Mac OS系统 可以不指定端口
hostname = [IP]:9090
#设置secretkey_path 的路径为 当前目录的data下
secretkey_path = ./data
```

#### 3.构建镜像

```shell
./install.sh
```

#### 4.上传镜像

（1）首先登录私有仓库，可以使用 admin 用户 ，也可以使用我们自己创建的具有上传权限的用户

```shell
docker login -u admin -p Harbor12345 127.0.0.1:9090
```

（2）要通过docker tag将该镜像标志为要推送到私有仓库，例如：

```shell
docker tag nginx:latest 127.0.0.1:9090/library/nginx:latest
```

（3）上传镜像

```shell
docker push 127.0.0.1:9090/library/nginx:latest
```

