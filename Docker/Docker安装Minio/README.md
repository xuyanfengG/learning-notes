#### 1.拉取镜像

```shell
docker pull minio/minio
```

#### 2.创建本地文件挂在目录

```shell
mkdir -p /data0/minio/data
mkdir -p /data0/minio/config
```

#### 3.启动容器【9000是minio通信端口，9001是web管理界面端口】

```shell
docker run -d -p 9000:9000 -p 9001:9001  --name minio -e "MINIO_ROOT_USER=mogu2018" -e "MINIO_ROOT_PASSWORD=mogu2018" -v /data0/minio/data:/data -v /data0/minio/config:/root/.minio minio/minio server  --console-address ":9001" --address ":9000" /data
```

访问地址：[http://IP:9001](http://ip:9001/)

