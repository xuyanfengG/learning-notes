#### 1.Source Connector

> 将数据库数据读取到kafka

```properties
connection.url #数据库连接地址
connection.user    #数据库用户名
connection.password  #数据库密码
table.whitelist #查询表白名单
connection.attempts  #数据库连接次数
numeric.precision.mapping  #是否根据进度判断数据类型
table.blacklist  #表黑名单
connection.backoff.ms  #数据库尝试连接间隔时间
schema.pattern  #查询表时使用的schema mode  #更新表时候用的模式,主要有以下4种模式.
    Bulk:每次都全部查询。
    timestamp：根据时间戳字段是否变化来检测数据是否增加或更新。
    incrementing ：用自增长字段来检测数据是否有增加，不能检测数据变化和删除。
    timestamp+incrementing ：根据时间戳和自增长字段来检查新增更新数据，根据自增长字段来标示唯一的流数据。
incrementing.column.name  #用来判断是否有新增数据的自增长字段名称，该字段值不允许为空值。
timestamp.column.name   #用来判断数据是否有新增和更新的时间戳字段，该字段值不允许为空值
validate.non.null    #设置是否检测数据库中自增长和时间戳字段不允许为空值，如果检查失败connector就停止启动。
 
query   #设置数据查询语句，如果设置了就不进行全表轮询，而是只是用此sql语句去提取数据
query.condition   #设置自定义查询条件，会拼接到where语句后面。（非自带，修改代码添加此参数）
poll.interval.ms  #数据轮询时间间隔，对数据库执行查询语句的时间间隔，此参数对数据库性能有影响
batch.max.rows  # connector每次轮询获取数据的最大数，默认100条，和connector获取数据的性能有关
table.poll.interval.ms #检查表是否有增加或删除的时间间隔。
topic.prefix  #使用普通查询时候以topic.prefix +表名 作为topic.自定义query语句时候以        topic.prefix作为topic，此处写目的库的表名（如果不知道搞不清楚生产到哪个主题，去kafka上验证一下）
 
table.types #设置要查询的表类型，默认为table,还可以设置view、system table。 timestamp.delay.interval.ms  #延迟转移时间间隔，可以延迟数据转移
```

#### 2.Sink Connector

> 负责将kafka数据插入到数据库

```properties
connection.url  #数据库连接URL
connection.user  #数据库连接用户名
connection.password  #数据库密码。
insert.mode # 插入数据模式 insert,upsert,update。本项目使用upsert模式，在没有相应主键数据时候直接插入，否则进行更新操作。
batch.size  #每次插入数据的最大数，和数据库性能有关
topics  #订阅的主题，也是目的数据库的表名
table.name.format  #插入表面格式化，默认为${topic}
pk.mode  #主键模式。None:不设置主键，kafka：使用kafka坐标作为主键，record_key: 使用record 的key字段作为主键，record_value：使用record 的value 字段作为主键。
 
pk.fields  #主键字段，以逗号分隔。项目中使用目的表的虚拟主键或者唯一约束字段。
fields.whitelist  #插入字段白名单，如果为空则所有字段都使用
auto.create  #是否自动创建目的表。
auto.evolve  #当表结构发生变化时候，是否自动修改目的表
max.retries   #失败重试次数 
retry.backoff.ms  #重试时间间隔
```

#### 3.启动

（1）启动Schema Registry

（2） 启动Source

```shell
./connect-standalone ../etc/schema-registry/connect-avro-standalone.properties /etc/kafka-connect-jdbc/ source-quickstart-sqlite.properties
```

（3）启动Sink

```shell
./connect-standalone ../etc/schema-registry/connect-avro-standalone.properties /etc/kafka-connect-jdbc/ sink-quickstart-sqlite.properties
```

#### 4.补充说明

（1）Kafka连接器序列化常见格式

JSON、Avro、Protobuf;、字符串分割（如CSV）

（2）Kafka连接器序列化常见转换器

* io.confluent.connect.avro.AvroConverter 
* org.apache.kafka.connect.storage.StringConverter 
* org.apache.kafka.connect.json.JsonConverter 
* org.apache.kafka.connect.converters.ByteArrayConverter 
* com.blueapron.connect.protobuf.ProtobufConverter

（3）模式

如果key和value包含模式，需要配置，可以分开配置

```properties
key.converter.schemas.enable=true	
value.converter.schemas.enable=true
```

