#### 1.查看当前docker文件路径

```shell
docker info
```

```shell
 ......
 Name: ecs-7b8a
 ID: ZBKY:C3DY:X3NA:IHIO:XLPB:VTMG:2EBX:FD5N:3GYS:4NHK:DEYQ:U5ZQ
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

文件路径为：/var/lib/docker

#### 2.停止所有正在运行的容器，停止docker服务

```shell
service docker stop
#
systemctl stop docker.socket
```

#### 3.文件迁移

```shell
cp -r  /var/lib/docker /data0/docker 
```

修改新路径的文件权限

```shell
chmod -R 777 /data0/docker 
```

#### 4.删除原有文件

```shell
rm -rf /var/lib/docker
```

#### 5.增加软连接

```shell
ln -s /data0/docker  /var/lib/docker
```

#### 6.启动docker服务

```shell
service docker start 
```

