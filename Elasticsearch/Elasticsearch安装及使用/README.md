## 一、源码安装及配置

### 1. 创建用户

```shell
adduser elasticsearch
passwd elasticsearch
chown -R elasticsearch elasticsearch-6.0.0
```

### 2. 配置

#### 2.1 修改elasticsearch.yml

```yaml
network.host: 192.168.2.72
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
# 如果系统内核版本过低，则在最底下添加
bootstrap.system_call_filter: false
```

#### 2.2 编辑/etc/sysctl.conf

```shell
vm.max_map_count=262144
```

或进行一次性修改

```shell
sysctl -w vm.max_map_count=262144
```

检查是否生效

```shell
sysctl -a | grep “vm.max_map_count”
```

#### 2.3 编辑/etc/security/limits.conf

```properties
elasticsearch soft nofile 65536
elasticsearch  hard nofile 65536
elasticsearch  soft nproc 4096
elasticsearch  hard nproc 4096
```

### 3. 启动

切换es用户

```shell
bin/elasticsearch -d
```

打开另一个终端，输入curl 'http://localhost:9200/?pretty'验证是否启动成功，若出现以下内容则表示成功

```json
{
  		"name" : "Tom Foster",
  		"cluster_name" : "elasticsearch",
 		"version" : {
    			"number" : "2.1.0",
    			"build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
    			"build_timestamp" : "2015-11-18T22:40:03Z",
    			"build_snapshot" : false,
    			"lucene_version" : "5.3.1"
  		},
  		"tagline" : "You Know, for Search"
}
```

## 二、Docker安装ES镜像及配置

#### 1 编辑/etc/sysctl.conf

```properties
vm.max_map_count=262144
```

或进行一次性修改

```shell
sysctl -w vm.max_map_count=262144
```

检查是否生效

```shell
sysctl -a | grep “vm.max_map_count”
```

#### 2 下载

```shell
docker pull elasticsearch:7.7.0
docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.7.0
```

### 3. 修改配置

```shell
# 进入容器
docker exec -it elasticsearch /bin/bash
# 编辑文件
vi config/elasticsearch.yml
```

添加以下内容

```yaml
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

重启容器

```shell
# 推出容器
exit
# 重启容器
docker restart 容器id
```

## 三、安装ElasticSearch-head

```shell
# 下载
docker pull mobz/elasticsearch-head:5
#创建容器
docker create --name elasticsearch-head -p 9100:9100 mobz/elasticsearch-head:5
#复制vendor.js到外部
docker cp [容器ID]:/usr/src/app/_site/vendor.js /usr/local/
#修改vendor.js
vim vendor.js
```

修改以下内容：

```js
// 6886行【vim命令:n跳转到n行】
contentType: "application/json;charset=UTF-8",
// 7573行
var inspectData = s.contentType === "application/json;charset=UTF-8" &&
docker cp /usr/local/vendor.js  [容器ID]:/usr/src/app/_site
```