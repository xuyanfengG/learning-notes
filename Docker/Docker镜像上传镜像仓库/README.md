## 一、上传阿里云镜像仓库

#### 1.上传步骤：

> 1. 登录阿里云镜像
> 2. 本地镜像标签化
> 3. 推送标签化镜像至阿里云镜像仓库
> 4. 删除本地标签化镜像

#### 2.命令如下：

> 1. 泰格esl：阿里云用户名
> 2. [registry.cn-beijing.aliyuncs.com](http://registry.cn-beijing.aliyuncs.com/)：阿里云镜像仓库地址
> 3. xuyanfeng：阿里云镜像仓库namespace
> 4. sharding-auth：阿里云xuyanfeng命名空间下的镜像仓库名

```shell
docker login --username=泰格esl registry.cn-beijing.aliyuncs.com
docker tag [ImageId] registry.cn-beijing.aliyuncs.com/xuyanfeng/sharding-auth:[镜像版本号]
docker push registry.cn-beijing.aliyuncs.com/xuyanfeng/sharding-auth:[镜像版本号]
docker rmi -f [标签化镜像name]:[标签化镜像版本号]
```

#### 3.从阿里云镜像仓库拉取镜像

```shell
docker pull registry.cn-beijing.aliyuncs.com/xuyanfeng/sharding-auth:[镜像版本号]
```

