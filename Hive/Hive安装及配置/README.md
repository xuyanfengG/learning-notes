#### 1.添加环境变量

```properties
export HIVE_HOME=/usr/local/hive
export PATH=$JAVA_HOME/bin:$PATH:$HADOOP_HOME/bin:$HIVE_HOME/bin
```

刷新配置文件

```shell
source /etc/profile
```

#### 2.修改配置文件

```shell
mv  hive-env.sh.template  hive-env.sh
mv  hive-default.xml.template  hive-site.xml
```

（1）修改hadoop的hadoop-env.sh(否则启动hive汇报找不到类的错误)

```properties
export HADOOP_CLASSPATH=.:$CLASSPATH:$HADOOP_CLASSPATH:$HADOOP_HOME/bin、
```

（2）修改$HIVE_HOME/bin的hive-config.sh，增加以下三行

```properties
export JAVA_HOME=/usr/local/jdk
export HIVE_HOME=/usr/local/hive
export HADOOP_HOME=/usr/local/hadoop
```

（3）修改$HIVE_HOME/conf/hive-site.xml

```xml
		<property>
			<name>javax.jdo.option.ConnectionURL</name>
			<value>jdbc:mysql://192.168.2.71:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
		</property>
		<property>
			<name>javax.jdo.option.ConnectionDriverName</name>
			<value>com.mysql.jdbc.Driver</value>
		</property>
		<property>
			<name>javax.jdo.option.ConnectionUserName</name>
			<value>root</value>
		</property>
		<property>
			<name>javax.jdo.option.ConnectionPassword</name>
			<value>123456</value>
		</property>
		<property>
          			<name>hive.metastore.uris</name>
          			<value>thrift://192.168.2.71:9083</value>
        		</property>

		<property>
  			<name>hive.metastore.warehouse.dir</name>
  			<value>/user/hive/warehouse</value>
		</property>
 		<!-- 配置hive用户认证，设置为NONE则跳过认证 -->
		 <property>
        			<name>hive.server2.authentication</name>
       			 <value>NONE</value>
		</property>
```

（4）上传mysql-connector-java-5.1.10.jar到$HIVE_HOME/lib

（5）登录mysql，创建hive库

```mysql
mysql>create database hive;
mysql>GRANT all ON hive.* TO root@'%' IDENTIFIED BY 'admin';
mysql>flush privileges;
mysql>set global binlog_format=‘MIXED‘;（按需设置）
# 把mysql的数据库字符类型改为latin1（按需设置）
```

（6）Hive的运行模式即任务的执行环境分为本地与集群两种，我们可以通过mapred.job.tracker 来指明设置方式：

```sql
hive > SET mapred.job.tracker=local
```

（7）初始化hive

```shell
schematool -dbType mysql -initSchema
```

（8）启动

```shell
# linux命令界面启动
bin/beeline 
!connect jdbc:hive2://localhost:10000
# 启动metastore（在store主服务器启动）
bin/hive --service metastore
# 进程方式启动(port:10000)：
bin/hive --service hiveserver2 >/dev/null 2>&1 &（供远程访问查询，同时10002为hiveserver2的web界面）
```

