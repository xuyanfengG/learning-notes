## 一、JAVA环境配置

### 1. 下载JDK安装包

### 2. 解压

```shell
tar -zxvf xxx.tar.gz
```

### 3. 编辑/etc/profile文件，添加以下内容

```properties
export JAVA_HOME=/data0/java/jdk1.8.0_111
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

### 4. 刷新环境变量

```shell
source /etc/profile
```

验证java版本信息

```shell
java -version
```

## 二、MAVEN配置

### 1. 修改配置文件settings.xml

（1）添加资源库

```xml
<localRepository>/data0/maven/mvn_repo</localRepository>
```

（2）修改默认jdk

```xml
<profile>
    <id>jdk-1.8</id>

    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>

    <properties> 
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

（3）添加阿里云仓库

```xml
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
```

### 2. 添加环境变量

```properties
JAVA_HOME=/data0/jdk180
MAVEN_HOME=/data0/maven/apache-maven-3.5.4
PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH
```

## 三、公钥与密钥

#### 1.生成密钥

root用户下

```shell
ssh-keygen
```

然后一路回车，密钥生成路径：[/root/.ssh/]

把公钥拷贝到远程服务器的 [/root/.ssh/authorized_keys] 文件中

```shell
ssh-copy-id -i /root/.ssh/id_rsa.pub [user]@[server]
```

## 四、更换yum源（/etc/yum.repos.d/）

### 1. centos6替换yum源

```shell
sed -i "s|enabled=1|enabled=0|g" /etc/yum/pluginconf.d/fastestmirror.conf
```

备份原有源，下载阿里源

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo https://www.xmpan.com/Centos-6-Vault-Aliyun.repo
```

清理及缓存yum包

```shell
yum clean all
yum makecache
```

## 五、清理缓存

```shell
sync;sync;

# 1. To free pagecache:
echo 1 > /proc/sys/vm/drop_caches
# 2. To free dentries and inodes:
echo 2 > /proc/sys/vm/drop_caches
# 3. To free pagecache, dentries and inodes:
echo 3 > /proc/sys/vm/drop_caches
```

