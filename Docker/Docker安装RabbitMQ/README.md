```shell
docker pull rabbitmq:management
docker run -di --name=mycloud_rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management
```

管理界面默认登录名密码：guest guest

