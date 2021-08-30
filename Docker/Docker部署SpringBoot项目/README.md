## 方式一：Docker打包镜像

#### 1.Springboot项目编译成jar包上传至服务器

#### 2.在当前文件目录下编写Dockerfile

```dockerfile
FROM java:8
MAINTAINER xyf
ADD ./demo-docker.jar /demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/demo.jar"]
```

各参数解释：

- from java:8  拉取一个jdk为1.8的docker image
- maintainer  作者
- demo-docker.jar 就是你上传的jar包，替换为jar包的名称
- demo.jar  是你将该jar包重新命名为什么名称，在容器中运行
- expose  该容器暴露的端口是多少，就是jar在容器中以多少端口运行
- entrypoint 容器启动之后执行的命令，java -jar demo.jar  即启动jar

#### 3.构建镜像

```shell
docker build -t demo:1.0 .
```

