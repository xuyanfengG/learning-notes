### 1. 修改hosts文件

```properties
0.0.0.0 localhost  host1
```

### 2. 添加环境变量

```properties
#export
CONFLUENT_HOME=XXX
#path中添加		
$CONFLUENT_HOME/bin
#classpath中添加		
$CONFLUENT_HOME/share/java/*
```

### 3. 配置Zookeeper

```properties
tickTime=2000 #时间单元，毫秒单位
dataDir=/runyi/confluent/zkdata #数据存储路径
#dataLogDir= 事务日志的存储路径
clientPort=2181 #zookeeper 客户端监听端口
initLimit=5 #followers 初始化时间
syncLimit=2 #followers 同步时间
maxClientCnxns=0 #最大client连接数，值为0的时候没有上限
server.1=192.168.2.224:2888:3888 #server1 地址，修改为自身地址
autopurge.snapRetainCount=3 #最近的快照保存数目
autopurge.purgeInterval=24 #快照自动清除时间间隔
```

a. 如果以confluent启动，则在/etc/kafka下

```shell
vi myid
# 输入0，保存退出编辑
```

在config_zookeeper()方法块最后一行，添加：

```shell
cp ${confluent_conf}/kafka/myid $confluent_current/zookeeper/data/
```

b. 若单独启动zk，则在zk的data目录下

```shell
vi myid
# 输入0，保存退出编辑
```

### 4. 配置Kafka

修改etc/kafka/server.properites

```properties
zookeeper.connect=192.168.2.224:2181
#broker.id=0 禁用ID，让kafka自动生成
broker.id.generation.enable=true
log.dirs=/runyi/confluent/kafka-logs #日志目录,修改为自身log目录  
listeners=PLAINTEXT://192.168.2.224:9092
advertised.listeners=PLAINTEXT://192.168.2.224:9092 #发布到zookeeper供客户端连接的地址
delete.topic.enable=true
confluent.metrics.reporter.bootstrap.servers=192.168.2.224:9092
confluent.metrics.reporter.topic.replicas=1
confluent.support.metrics.enable=true
confluent.support.customer.id=anonymous
group.initial.rebalance.delay.ms=0
```

### 5. 配置Schema Registry

修改etc/schema-registry/schema-registry.properites 文件

```properties
listeners=0.0.0.0:8081
kafkastore.connection.url=192.168.2.224:2181
kafka.bootstrap.servers=PLAINTEXT://192.168.2.224:9092
debug=false
kafkastore.topic=_schemas
```

### 6. 配置Kafka-Connect

修改etc/schema-registry/connect-avro-distributed.properties.properites 文件（这是集群默认，单机配置stan............，没有的不用配置）

```properties
group.id=connect-cluster
bootstrap.servers=192.168.2.224:9092
key.converter=io...
key.converter.schema......=http://192.168.2.224:8081
value.converter=io.........
value.converter.schema.....=http://192.168.2.224:8081
key.converter.schemas.enable=false #防止key中文乱码
value.converter.schemas.enable=true
config.storage.topic=connect-configs
offset.storage.topic=connect-offsets
status.storage.topic=connect-statuses
config.storage.replication.factor=3
offset.storage.replication.factor=3
status.storage.replication.factor=3
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter.schemas.enable=false
internal.value.converter.schemas.enable=false
rest.port=8083
rest.advertised.port=8083
```

### 7. 配置Kafka Rest

```properties
listener=0.0.0.0:8082
schema.registry.url=http://192.168.2.224:8081
zookeeper.connect=192.168.2.224:2181
bootstrap.servers=PLAINTEXT://192.168.2.224:9092
consumer.....=io....
producer.......=io....
```

### 8. 配置Ksql

修改etc/ksql/ksql-server.properties

```properties
bootstrap.servers=192.168.2.224:9092
ksql.schema.registry.url=http://192.168.2.224:8081
ksql.service.id=default_
listeners=http://0.0.0.0:8088
ksql.extension.dir=/data0/confluent-5.4.0/etc/ksql/ext
```

### 9.启动

```
# zookeeper
nohup bin/zookeeper-server-start etc/kafka/zookeeper.properties >zkout.file 2>&1 &
# kafka
nohup bin/kafka-server-start etc/kafka/server.properties >kafkaout.file 2>&1 &
# schema registry
nohup bin/schema-registry-start etc/schema-registry/schema-registry.properties >schemaout.file 2>&1 &
# kafka rest
nohup bin/kafka-rest-start etc/kafka-rest/kafka-rest.properties >restout.file 2>&1 &
# ksql
nohup bin/ksql-server-start etc/ksql/ksql-server.properties >ksqlout.file 2>&1 &	
# connect(单机)
nohup bin/connect-standalone etc/schema-registry/connect-avro-standalone.properties etc/kafka-connect-jdbc/sink-quickstart-sqlite-hour.properties etc/kafka-connect-jdbc/sink-quickstart-sqlite-24hour.properties >connectstanout.file 2>&1 &
```

### 10.Ksql使用

（1）shell操作

进入ksql终端：

```shell
bin/ksql http://localhost:8088
```

操作：

```sql
# 查看流、表，持久流
show streams;
show tables;
show queries;
# 查看流、表，持久流的详细信息
describe xxx;
# 让KSQL从该主题的开始展示数据（而不是交替的当前时间点）
SET 'auto.offset.reset' = 'earliest'; 
# 创建流
CREATE STREAM text_str (name varchar, num bigint) WITH (kafka_topic='text_data', value_format='JSON');
# 创建表
CREATE TABLE users (registertime BIGINT, gender VARCHAR, regionid VARCHAR, userid VARCHAR) WITH (kafka_topic='users', value_format='JSON', key = 'userid')
# 创建持久流/表（即从流到流或从流到表）
CREATE STREAM/TABLE pageviews_intro AS SELECT * FROM pageviews WHERE pageid < 'Page_20' ;
# 删除流、表
drop stream/table xxx;
# 删除持久流/表
# 先停止 
TERMINATE xxx;
# 再删除 
drop stream/table xxx;
# 查询流、表数据（创建的流可以直接同步kafka相应的数据）
select * from xxx emit changes;
```

（2）序列化格式

KSQL当前支持序列化格式：

* DELIMITED支持逗号分隔的值。请参阅下面的DELIMITED。
* JSON支持JSON值。
* AVRO支持AVRO序列化值。
* KAFKA支持使用标准Kafka序列化器序列化的基元。

KSQL支持以下原始数据类型：

* BOOLEAN 
* INTEGER （或INT ）
* BIGINT 
* DOUBLE 
* VARCHAR（或STRING）