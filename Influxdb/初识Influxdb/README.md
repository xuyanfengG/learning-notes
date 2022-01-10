#### 1. Influxdb介绍

Influxdb是一种时序数据库（TSDB）

##### 1.1 存储引擎

* **cache**：插入数据时，先往 cache 中写入再写入wal中，可以认为 cache 是 wal 文件中的数据在内存中的缓存，cache 中的数据并不是无限增长的，有一个 maxSize 参数用于控制当 cache 中的数据占用多少内存后就会将数据写入 tsm 文件。如果不配置的话，默认上限为 25MB
* **wal**：预写日志，对比MySQL的 binlog，其内容与内存中的 cache 相同，作用就是为了持久化数据，当系统崩溃后可以通过 wal 文件恢复还没有写入到 tsm 文件中的数据，当 InfluxDB 启动时，会遍历所有的 wal 文件，重新构造 cache。
* **tsm file**：每个 tsm 文件的大小上限是 2GB。当达到 cache-snapshot-memory-size,cache-max-memory-size 的限制时会触发将 cache 写入 tsm 文件。
* **compactor**：主要进行两种操作，一种是 cache 数据达到阀值后，进行快照，生成一个新的 tsm 文件。另外一种就是合并当前的 tsm 文件，将多个小的 tsm 文件合并成一个，减少文件的数量，并且进行一些数据删除操作。 这些操作都在后台自动完成，一般每隔 1 秒会检查一次是否有需要压缩合并的数据。

##### 1.2 存储目录

- **meta**：用于存储数据库的一些元数据，meta 目录下有一个 meta.db 文件；
- **wal**：目录存放预写日志文件，以 .wal 结尾；
- **data**：目录存放实际存储的数据文件，以 .tsm 结尾。

#### 2. 相关概念

* **database**：数据库
* **measurement**：数据库中的表
* **point**：表中的行，由时间戳（time）、标签（tags）、数据（fields）三部分组成
  * **time**：数据记录的时间，是主索引（自动生成）
  * **tags**：有索引的属性，只支持字符型
    * 只能使用Tag进行Group
    * 只能使用Tag进行正则表达式操作
  * **fields**：没有索引的属性，可支持多种数据类型
    * 只能使用Field进行函数操作，例如sum()
    * 只能使用Field进行比较操作
    * 如果需要使用int,float,boolean类型进行存储,只能使用Field
* **series**：一般由retention policy、measurement、tag set三部分组成
* **retention policy**：存储策略，可以设置数据保存时间，默认是autogen，即永久保存
* **shard**：数据片，每一个存储策略下有多个数据片，每一个数据片存储一个指定时间段内的数据，且不重复。例如 7点-8点 的数据落入shard0中，8点-9点的数据则落入shard1中。每一个shard都对应一个底层的tsm存储引擎，有独立的cache、wal、tsm file
* **shard duration**：决定每个shard group的时间跨度，具体由retention policy的【SHARD DURATION】决定
* **shard group**：按照time、retention policy进行组织。每个包含数据的retention policy至少有一个关联的shard group。给定的shard group包含时间区间内所有shard数据。 每个shard group跨越的间隔是shard duration

#### 3.sql

一条记录的存储格式：

```sql
<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```

常用sql：

```sql
-- 连接
influx
-- 登录
auth
-- 查看所有的数据库
show databases;
-- 使用特定的数据库
use database_name;
-- 查看所有的measurement
show measurements;
-- 查询10条数据
select * from measurement_name limit 10;
-- 数据中的时间字段默认显示的是一个纳秒时间戳，改成可读格式
precision rfc3339; -- 之后再查询，时间就是rfc3339标准格式
-- 或可以在连接数据库的时候，直接带该参数
influx -precision rfc3339
-- 查看一个measurement中所有的tag key 
show tag keys
-- 查看一个measurement中所有的field key 
show field keys
-- 查看一个measurement中所有的保存策略(可以有多个，一个标识为default)
show retention policies;
```

#### 4.Java应用

```xml
        <!-- influxdb-->
        <dependency>
            <groupId>org.influxdb</groupId>
            <artifactId>influxdb-java</artifactId>
            <version>${influxdb.version}</version>
        </dependency>
```

```java
@Configuration
public class InfluxConfig {

    @Value("${spring.influx.url}")
    private String url;
    @Value("${spring.influx.user}")
    private String user;
    @Value("${spring.influx.password}")
    private String password;
    @Value("${spring.influx.database}")
    private String database;
    @Value("${spring.influx.retentionPolicyName}")
    private String retentionPolicyName;
    @Bean
    public InfluxDB initInflux(){
        InfluxDB influxDB = InfluxDBFactory.connect(url,user,password);
        influxDB.setDatabase(database);
        influxDB.setRetentionPolicy(retentionPolicyName);
        influxDB.enableBatch(
                BatchOptions.DEFAULTS
                        .threadFactory(runnable -> {    // 设置批处理线程工厂
                            Thread thread = new Thread(runnable);
                            thread.setDaemon(true);     // 设置为守护线程。JVM在异常的或主线程完成的情况下会自动关闭，会影响该线程池中正在
                            // 运行的线程，导致无法正常运行和关闭
                            return thread;
                        })
                        .jitterDuration(3000)           // 批处理抖动时间
        ).enableGzip();
        // 手动添加关闭influxDB的钩子，当JVM关闭前会首先关闭该钩子
        Runtime.getRuntime().addShutdownHook(new Thread(influxDB::close));
        return influxDB;
    }

    @Bean
    public InfluxDBResultMapper getInfluxDBResultMapper(){
        return new InfluxDBResultMapper();
    }
```

```java
    @Test
    void batchWrite(){
        // influxDB.write(a1)为异步，batchPoints为同步
        Point a1 = null;
        for (int i = 0; i < 10; i++){
            a1 = Point.measurement("testTable")
                    .time(System.currentTimeMillis(), TimeUnit.MILLISECONDS)
                    .tag("gatewayId", "110"+i)
                    .tag("deviceId", "2210"+i)
                    .tag("diId", "5210"+i)
                    .tag("pubTopic", "1011"+i)
                    .addField("processedData", i)
                    .addField("originalData", i)
                    .addField("reportedOn", "2021-11-04 12:00:01")
                    .addField("tamp", "2021-11-04 12:00:02")
                    .build();
            influxDB.write(a1);
        }
    }
        @Test
    void read(){
        // 利用POJO读取
        QueryResult queryResult1 = influxDB.query(new Query("SELECT * FROM testTable"));
        log.info(queryResult1.getResults().toString());
    }
```

