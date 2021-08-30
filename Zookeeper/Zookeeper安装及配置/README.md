### 1. 下载地址

https://downloads.apache.org/zookeeper/

### 2. 上传文件并解压

```shell
tar -zxvf xxxx.tar.gz
```

### 3. 修改配置文件

```shell
vim conf
```

修改以下配置项：

```properties
# 数据路径
dataDir=/home/zookeeper/tmp/data
# 日志路径
dataLogDir=/home/zookeeper/tmp/logs
# 端口号
clientPort=2181
```

### 4. 启动&停止

```shell
# 启动
./zkServer.sh start
# 停止
./zkServer.sh stop
# 查看状态
./zkServer.sh status
```

