> 将kafka中avro格式数据导入hdfs并用hive外表关联

#### 1.编辑**connect-avro-standalone.properties**文件，以下配置需要格外注意，其他配置根据实际情况配置即可

> 因为hive支持avro格式数据，所以将kafka数据转为avro，再同步hdfs和hive

```properties
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=io.confluent.connect.avro.AvroConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=true
```

#### 2.下载kafka-connect-hdfs连接器，将jar包放在share/java/kafka-connect-hdfs，将驱动器配置文件放到etc/kafka-connect-hdfs下

修改驱动器配置文件：

```properties
name=hdfs-sink1
connector.class=io.confluent.connect.hdfs.HdfsSinkConnector
tasks.max=1
topics=RAONECLOUD_TABLE_HOUR_1
hdfs.url=hdfs://192.168.2.224:9000/kafka
flush.size=3
format.class=io.confluent.connect.hdfs.avro.AvroFormat

#同步hive操作
hive.integration=true
#需要在服务端启动metastore
hive.metastore.uris=thrift://192.168.2.224:9083
schema.compatibility=BACKWARD
```

#### 3.文件赋权

```shell
hadoop fs -chmod -R 777 /tmp
```

