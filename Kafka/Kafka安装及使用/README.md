### 1. 下载地址

http://kafka.apache.org/downloads.html

### 2. 上传及解压

```shell
tar -zxvf xxx.tar.gz
```

### 3. 修改配置文件

```shell
vim config/server.properties
```

修改以下配置项：

```properties
# 日志路径
log.dirs=/home/kafka/kafka-logs
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
# zookeeper地址
zookeeper.connect=localhost:2181
# 配置内网访问地址
listeners=PLAINTEXT://192.168.2.97:9092
# 配置外网访问地址（按需配置）
advertised.listeners=PLAINTEXT://192.168.2.97:9092
# 配置外网访问安全协议（按需配置）
listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
```

### 4. 启动&停止

```shell
# 启动
bin/kafka-server-start.sh config/server.properties &
# 停止
bin/kafka-server-stop.sh config/server.properties
```

## 二、Kafka使用

```shell
# 启动
nohup bin/kafka-server-start.sh config/server.properties & 
# 关闭
bin/kafka-server-stop.sh
# 查询topic
bin/kafka-topics --zookeeper localhost:2181 --list
# 查询指定topic明细
bin/kafka-topics --zookeeper localhost:2181 --desc --topic RAONECLOUD_TABLE_HOUR
# 或 
bin/kafka-topics --zookeeper localhost:2181 --topic RAONECLOUD_TABLE_HOUR --describe
# 删除topic
bin/kafka-topics  --delete --zookeeper localhost:2181 --topic raonecloud
# 创建topic
bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic raonecloud
# 消费数据
bin/kafka-console-consumer --bootstrap-server 192.168.2.71:9092 --topic raonecloud --from-beginning
									--offset 10 --partition 0
# 插入数据
bin/kafka-console-producer --broker-list 192.168.2.71:9092 --topic raonecloud
```