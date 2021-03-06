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

## 二、安装第三方镜像仓库：Harbor

> docker 官方提供的私有仓库 registry，用起来虽然简单 ，但在管理的功能上存在不足。 Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，harbor使用的是官方的docker registry(v2命名是distribution)服务去完成。harbor在docker distribution的基础上增加了一些安全、访问控制、管理的功能以满足企业对于镜像仓库的需求。
>
> 安装Harbor前需要安装docker和docker-compose

#### 1.下载harbor（以2.3.2为例）

https://github.com/goharbor/harbor/releases

https://storage.googleapis.com/harbor-releases/

#### 2.解压并修改配置文件（如果harbor已经启动，修改完后需执行./prepare）

```yaml
# 修改IP
hostname: 192.168.0.63:9800
# 端口号
http:
  port: 80
# 屏蔽掉https配置（如果需要https，则配置）
# 修改admin账户密码
harbor_admin_password: 123456
# 修改harbor数据库的默认密码
database:
  password: 123456
# 修改数据存储位置
data_volume: /data0/harbor/data
# 修改日志路径
log:
  local:
    location: /data0/harbor/log
```

编辑并保存docker-compose.yml文件（如果harbor已经启动，需要重新编辑修改）：

```yaml
proxy:
    image: goharbor/nginx-photon:v2.1.0
    container_name: nginx
    restart: always
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - NET_BIND_SERVICE
    volumes:
      - ./common/config/nginx:/etc/nginx:z
      - type: bind
        source: ./common/config/shared/trust-certificates
        target: /harbor_cust_cert
    networks:
      - harbor
    dns_search: .
    ports:
      #此处原本为80:8080，将80端口修改为9800端口
      - 9800:8080
    depends_on:
      - registry
      - core
      - portal
      - log
```

#### 3.构建镜像（加上参数支持helm）

```shell
./install.sh --with-chartmuseum
```

Harbor依赖的镜像及启动服务如下：

```shell
$ docker-compose ps
       Name                     Command               State                                Ports                               
------------------------------------------------------------------------------------------------------------------------------
harbor-adminserver   /harbor/harbor_adminserver       Up                                                                       
harbor-db            docker-entrypoint.sh mysqld      Up      3306/tcp                                                         
harbor-jobservice    /harbor/harbor_jobservice        Up                                                                       
harbor-log           /bin/sh -c crond && rm -f  ...   Up      127.0.0.1:1514->514/tcp                                          
harbor-ui            /harbor/harbor_ui                Up                                                                       
nginx                nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp 
registry             /entrypoint.sh serve /etc/ ...   Up      5000/tcp 
------------------------------------------------------------------------------------------------------------------------------
# 重启
$ docker-compose down
$ docker-compose up -d
```

#### 4.Harbor管理界面

登陆信息是（密码为上文配置文件中配置）：admin/123456

创建名称为**raone**的项目，设置不公开（如果公开命令行用户不需要登陆即可拉取镜像）

#### 4.docker配置（由于没有配置https，所以需要配置http）【所有需要登录镜像仓库的docker都需要配置】

```shell
vim /usr/lib/systemd/system/docker.service
#需要修改的地方
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=192.168.0.63:9800
```

```shell
#重新加载配置（生产环境慎用）
systemctl daemon-reload
#重启Docker服务
systemctl restart docker
```

#### 4.上传镜像

（1）首先登录私有仓库，可以使用 admin 用户 ，也可以使用我们自己创建的具有上传权限的用户

```shell
docker login 192.168.0.63:9800
Username: admin
Password: 123456
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

（2）要通过docker tag将该镜像标志为要推送到私有仓库，例如：

```shell
docker tag java:8 192.168.0.63:9800/raone/java:8
```

（3）上传镜像

```shell
docker push 192.168.0.63:9800/raone/java:8
```

（4）拉取镜像

```shell
docker pull 192.168.0.63:9800/raone/java:8
```

