## 一、安装及配置

### 1. 下载

（1）下载安装包：https://src.fedoraproject.org/repo/pkgs/haproxy

（2）安装依赖包

```shell
 yum install gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

（3）上传安装包至【/usr/local/src/】，解压安装

```shell
cd /usr/local/src/
tar -zxvf xxx.tar.gz         #解压
mv xxx haproxy               #重命名

cd /usr/local/src/haproxy
make TARGET=linux26                 #此处需要指定TARGET参数才可安装，
make install PREFIX=/usr/local/haproxy      #PREFIX指定安装路径
```

（4）新建配置文件**haproxy.cfg**

```properties
global
        daemon
        nbproc 1
        pidfile /var/run/haproxy.pid
        # 工作目录
        chroot /usr/local/etc/haproxy

defaults
        log 127.0.0.1 local0 info#[err warning info debug]
        mode http                #默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
        retries 3                #两次连接失败就认为是服务器不可用，也可以通过后面设置
        option redispatch        #当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
        option abortonclose      #当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
        option dontlognull       #日志中不记录负载均衡的心跳检测记录
        maxconn 4096             #默认的最大连接数
        timeout connect 5000ms   #连接超时
        timeout client 30000ms   #客户端超时
        timeout server 30000ms   #服务器超时
        #timeout check 2000      #=心跳检测超时

######## 监控界面配置 #################
listen admin_status
        # 监控界面访问信息
        bind 0.0.0.0:8888
        mode http
        # URI相对地址
        stats uri /dbs
        # 统计报告格式
        stats realm Global\ statistics
        # 登录账户信息
        stats auth admin:admin
########frontend配置##############

######## mysql负载均衡配置 ###############
listen proxy-mysql
        bind 0.0.0.0:3320
        mode tcp
        # 负载均衡算法
        # static-rr 权重, leastconn 最少连接, source 请求IP, 轮询 roundrobin
        balance roundrobin
        # 日志格式
        option tcplog
        # 在 mysql 创建一个没有权限的haproxy用户，密码为空。 haproxy用户
        # create user 'haproxy'@'%' identified by ''; FLUSH PRIVILEGES;
        # 在这里是监控的sharding proxy，故不需要配置
        # option mysql-check user haproxy
        # 这里是sharding proxy代理的地址，由于配置的是轮询roundrobin，weight 权重其实没有生效
        server MYSQL_1 192.168.2.22:3310 check weight 1
        server MYSQL_2 192.168.2.22:3311 check weight 1
        # 使用keepalive检测死链
        # option tcpka
#########################################
```

（5）启动

a. docker启动

```shell
docker run -d --name haproxy1 -p 8888:8888 -p 3320:3320 -v /mnt/c/Users/T_m_T/haproxy:/usr/local/etc/haproxy haproxy:1.9.6 
```

b. 进程启动

```shell
/usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/etc/haproxy.cfg 
```

（6）web监控：http://[ip]:[PORT]/dbs