# docker-swarm搭建Redis哨兵集群

## 环境介绍

| ip             | **label** | **redis port** | **sentinel port** |
| -------------- | --------- | -------------- | ----------------- |
| 172.51.217.208 | master    | 6379           | 26379             |
| 172.51.217.207 | slave1    | 6379           | 26379             |
| 172.51.217.206 | slave2    | 6379           | 26379             |



### 防火墙

1. 防火墙状态

```shell
systemctl status firewalld
```

2. 停止/关闭防火墙

```shell
systemctl stop/disable firewalld
```





## Master

创建网络

```shell
docker network create --driver overlay redis_cluster
```

**创建文件夹**
mkdir -pv /redis/{bin,conf,data,logs}

**配置文件**
vi /redis/bin/docker-compose.yml

```shell
version: '3.4'
services:
  master:
    image: redis
    restart: always
    command: redis-server /etc/redis/redis.conf
    volumes:
      - /redis/data:/data
      - /redis/conf/redis-master.conf:/etc/redis/redis.conf:ro
      - /redis/logs:/var/log/redis
    ports:
    - target: 6379
      published: 6379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==master] # 条件约束 master指每台机器的label,docker node ls查看其hostname, 再通过docker node inspect {hostname}查其node.labels.role
    networks:
      - redis_cluster

  slave1:
    image: redis
    restart: always
    command: redis-server /etc/redis/redis.conf
    volumes:
      - /redis/data:/data
      - /redis/conf/redis-slave-1.conf:/etc/redis/redis.conf:ro
      - /redis/logs:/var/log/redis
    depends_on:
      - master
    ports:
    - target: 6379
      published: 6379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==slave1]  
    networks:
      - redis_cluster

  slave2:
    image: redis
    restart: always
    command: redis-server /etc/redis/redis.conf
    volumes:
      - /redis/data:/data
      - /redis/conf/redis-slave-2.conf:/etc/redis/redis.conf:ro
      - /redis/logs:/var/log/redis
    depends_on:
      - master
    ports:
    - target: 6379
      published: 6379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==slave2]
    networks:
      - redis_cluster

  sentinel1:
    image: redis
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    depends_on:
      - master
      - slave1
      - slave2
    ports:
    - target: 26379
      published: 26379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==master]
    networks:
      - redis_cluster
    volumes:
      - /redis/conf/sentinel1.conf:/usr/local/etc/redis/sentinel.conf

  sentinel2:
    image: redis
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    depends_on:
      - master
      - slave1
      - slave2
    ports:
    - target: 26379
      published: 26379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==slave1]
    networks:
      - redis_cluster
#    network_mode: "host"
    volumes:
      - /redis/conf/sentinel2.conf:/usr/local/etc/redis/sentinel.conf

  sentinel3:
    image: redis
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: always
    depends_on:
      - master
      - slave1
      - slave2
 #   network_mode: "host"
    ports:
    - target: 26379
      published: 26379
      protocol: tcp
      mode: host
    deploy:
      placement:
        constraints: [node.labels.role==slave2]
    networks:
      - redis_cluster
    volumes:
      - /redis/conf/sentinel3.conf:/usr/local/etc/redis/sentinel.conf

networks:
  redis_cluster:
    external:
      name: redis_cluster
```

创建启动命令

vi /redis/bin/startup.sh

```shell
#!/bin/bash

docker stack deploy -c docker-compose.yml redis_cluster
```

vi /redis/bin/shutdown.sh

```shell
#!/bin/bash

docker stack rm redis_cluster
```



vi /redis/conf/redis-master.conf

```shell
port 6379
bind 0.0.0.0
logfile "redis-master.log"
dbfilename "dump-master.rdb"
appendfilename appendonly-master.aof
rdbcompression yes
appendonly yes
requirepass 1234
slave-announce-ip 172.16.0.106 #{master ip}
slave-announce-port 6379
slave-read-only no

```

vi /redis/conf/sentinel1.conf

```shell
## 配置哨兵监控的master节点信息
# 默认配置sentinel monitor mymaster 127.0.0.1 6379 2
# mymaster是默认的，也可以自己命名，但是在自己命名之后需要将redis-sentinel.conf文件中所有的mymaster都要替换，否则在启动# # 哨兵时就会报错
# 127.0.0.1是主从模式下主节点的IP
# 6379是主从模式下主节点的PORT
# 2表示的是至少有两个哨兵认为主节点主观下线之后才会将主节点客观下线之后会重新选举新的主节点，下线后的主节点在重启之后会变成新# # 的子节点
port 26379
sentinel monitor mymaster 172.51.217.208 6379 2 
```



## Slave-1节点

**创建文件夹**

```
mkdir -pv /redis/{conf,data,logs}
```

vi /redis/conf/redis-slave-1.conf

```shell
port 6379
bind 0.0.0.0
logfile "redis-master.log"
dbfilename "dump-master.rdb"
appendfilename appendonly-master.aof
rdbcompression yes
appendonly yes
requirepass 1234
slaveof 172.51.217.208 6379
masterauth 1234
slave-announce-ip 172.51.217.207
slave-announce-port 6379
slave-read-only no

```

vi /redis/conf/sentinel2.conf

```shell
port 26379
sentinel monitor mymaster 172.51.217.208 6379 2 
```

## Slave-2节点

**创建文件夹**
mkdir -pv /redis/{conf,data,logs}
**配置文件**
vi /redis/conf/redis-slave-2.conf

```shell
port 6379
bind 0.0.0.0
logfile "redis-master.log"
dbfilename "dump-master.rdb"
appendfilename appendonly-master.aof
rdbcompression yes
appendonly yes
requirepass 1234
slaveof 172.51.217.208 6379
masterauth 1234
slave-announce-ip 172.51.217.206
slave-announce-port 6379
slave-read-only no

```

vi /redis/conf/sentinel3.conf

```shell
port 26379
sentinel monitor mymaster 172.51.217.208 6379 2 
```



# shell脚本简化

