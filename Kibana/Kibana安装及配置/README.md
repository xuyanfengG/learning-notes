### 1. 下载

### 2. 配置

编辑kibana.yml

```yaml
#配置本机ip
server.host: "192.168.252.129"
#配置es集群url
elasticsearch.hosts: ["http://192.168.252.129:9200"]
```

### 3. 启动

```shell
bin/kibana
```

验证启动：http:// [ip]:5601

