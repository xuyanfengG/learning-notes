### 1. 官网下载

[概览 :: ShardingSphere (apache.org)](https://shardingsphere.apache.org/document/legacy/4.x/document/cn/overview/)

### 2. 配置

#### 2.1 在文件根目录下新建文件夹ext-lib，将mysql驱动包复制到该目录下

#### 2.2 server.yaml（配置注册中心、权限管理和基础配置）

```yaml
#orchestration:
#  orchestration_ds:
#    orchestrationType: registry_center,config_center,distributed_lock_manager
#    instanceType: zookeeper
#    serverLists: localhost:2181
#    namespace: orchestration
#    props:
#      overwrite: false
#      retryIntervalMilliseconds: 500
#      timeToLiveSeconds: 60
#      maxRetries: 3
#      operationTimeoutMilliseconds: 500
#
authentication:
  users:
    root:
      password: root
#    sharding:
#      password: sharding 
#      authorizedSchemas: sharding_db

props:
  max.connections.size.per.query: 1
  acceptor.size: 16  # The default value is available processors count * 2.
  executor.size: 16  # Infinite by default.
  proxy.frontend.flush.threshold: 128  # The default value is 128.
    # LOCAL: Proxy will run with LOCAL transaction.
    # XA: Proxy will run with XA transaction.
    # BASE: Proxy will run with B.A.S.E transaction.
  proxy.transaction.type: LOCAL
  proxy.opentracing.enabled: false
  proxy.hint.enabled: false
  query.with.cipher.column: true
  sql.show: false
  allow.range.query.with.inline.sharding: true
```

#### 2.3 读写分离配置（版本4.1.0）

```yaml
schemaName: ms_ds_1

dataSources:
  master_ds:
    url: jdbc:mysql://localhost:3306/master?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 5
  slave_ds_0:
    url: jdbc:mysql://localhost:3307/master?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 5


masterSlaveRule:
  name: ms_ds
  masterDataSourceName: master_ds
  slaveDataSourceNames:
    - slave_ds_0
```

#### 2.4 分片（按时间分表时分片列类型最好是字符型，版本5.0.0已支持配置按时间分表）

```yaml
schemaName: sharding_db

dataSources:
  ds_0:
    url: jdbc:mysql://localhost:3308/test0?serverTimezone=UTC&useSSL=false
    username: root
    password: 123456
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
    minPoolSize: 1
    maintenanceIntervalMilliseconds: 30000
rules:
- !SHARDING
  tables:
    t_msg:
      actualDataNodes: ds_0.t_msg_${202104..202106}
      tableStrategy:
        standard:
          shardingColumn: createtime
          shardingAlgorithmName: t_order_inline
  bindingTables:
    - t_msg
  defaultTableStrategy:
    none:
  
  shardingAlgorithms:
    t_order_inline:
      type: INTERVAL
      props:
        datetime-pattern: 'yyyy-MM-dd HH:mm:ss'
        datetime-lower: '2021-01-01 00:00:00'
        # datetime-upper:
        sharding-suffix-pattern: 'yyyyMM'
        datetime-interval-amount: 1
        datetime-interval-unit: MONTHS

  
#  keyGenerators:
#    snowflake:
#      type: SNOWFLAKE
#      props:
#        worker-id: 123
```