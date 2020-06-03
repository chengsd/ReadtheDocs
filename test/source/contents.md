# 平台部署指南

## 域名规划

| 服务     | 域名                                                         |
| -------- | ------------------------------------------------------------ |
| 代码审计mysql | mysql.codesec.asi                                       |
| rabbitmq | rabbitmq.asmp.asi                                       |
| mongodb  | mongodb.codesec.asi                                    |
| zipkin   | zipkin.server.asmp.asi                          |
| eureka   | eureka.springcloud.asmp.asi               |
| 用户中心 | asuc.asmp.asi |
| 网关     | zuul.springcloud.asmp.asi |
| 能力平台后端程序 | business.platform.codesec.asi |
| Admin | admin.springcloud.asmp.asi |
| 用户中心mysql | mysql.asmp.asi|
| redis | redis.asmp.asi |
| codepecker引擎 |codepecker.codesec.asi|

## host文件

编辑host文件：

```
vi /etc/hosts
```

host文件格式示例：

```
10.236.77.27 mysql.codesec.asi 
10.236.77.32 rabbitmq.asmp.asi 
10.236.77.28 mongodb.codesec.asi 
10.236.77.21 zipkin.server.asmp.asi 
10.236.77.21 eureka.springcloud.asmp.asi 
10.236.77.32 asuc.asmp.asi
10.236.77.21 zuul.springcloud.asmp.asi
10.236.77.32 business.platform.codesec.asi
10.236.77.21 admin.springcloud.asmp.asi
10.236.77.30 mysql.asmp.asi
10.236.77.32 redis.asmp.asi
```

---
## docker离线部署



将docker-19.03.0.tgz解压到docker文件夹中，运行命令将文件拷贝到/usr/bin/中。

```shell
chmod +x docker/*
cp -r docker/* /usr/bin/
```

创建docker.service文件。

```
cat >/etc/systemd/system/docker.service
```

在docker.service文件中写入以下内容：

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target
```

通过systemctl启动docker，将其设置为开机自启动并查看docker状态。

```
systemctl enable docker
sysetmctl start docker
systemctl status docker
```

若观察到状态为active，说明运行成功。若无法运行，则升级内核，再度尝试。

---
## mysql 安装



mysql请安装5.7版本，以免用户中心后端无法连接。

创建mysql文件夹并将mysql-5.7.30-1.el7.x86_64.rpm-bundle.tar中的文件解压到文件夹中。

通过 ``` rpm -qa | grep mariadb```查看系统是否安装mariadb，若有，则通过```rpm -e``` 卸载。

依次运行：

```
rpm -ivh mysql-community-common-5.7.30-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.30-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.30-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.30-1.el7.x86_64.rpm
mysqld --initialize
chown mysql:mysql /var/lib/mysql -R
systemctl start mysqld
```

通过```cat /var/log/mysqld.log | grep password```查看生成的密码，在登陆后修改密码。

创建misas用户并赋予权限：

```
create user 'misas'@'%' identified by 'JdMm,ncbd.9876..';
grant all privileges on *.* to 'misas'@'%' identified by 'JdMm,ncbd.9876..';
flush privileges;
```

对于用户中心所用的mysql，需要创建misas_asuc数据库并导入misas_asuc_20200525.sql文件。

对于用户中心所用的mysql，需要创建misas数据库，```ALTER DATABASE misas CHARACTER SET `utf8mb4;```设置编码格式为utf8mb4，并导入misas-20200529.sql,misas_ct_two_level_details.sql,misas_ct_task_statistics.sql,misas_ct_misas_second_grade_rule.sql,misas_ct_misas_first_grade_rule.sql,misas_ct_codesec_config_info.sql文件。



---

## mongodb安装

上传mongodb的rpm安装包，进行安装。

```
rpm -ivh mongodb-org-server-4.0.5-1.el7.x86_64.rpm
rpm -ivh mongodb-org-shell-4.0.5-1.el7.x86_64.rpm 
rpm -ivh mongodb-org-tools-4.0.5-1.el7.x86_64.rpm
```

修改/etc/mongod.conf中的bindIP为0.0.0.0。使用```systemctl start mongod```启动mongodb。

---

## 引擎控制器部署

### 上线准备

1. 新建文件目录 /home/misas/codepecker/download,/home/misas/codepecker/logs,/home/misas/codepecker/db

2. 将数据文件codepecker.db置于/home/misas/codepecker/db目录下

   

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i codepecker.tar
```

### docker启动：

```
docker run -d -e "SPRING_PROFILES_ACTIVE=prod" -v /home/misas/docker/codepecker/logs/:/home/misas/codepecker/logs/ -v /home/misas/docker/codepecker/download/:/home/misas/codepecker/download/ -v /home/misas/docker/codepecker/db/:/home/misas/codepecker/db/  --net=host --name codepecker_client 2ed
```



---

## 代码审计后端程序code-sec 部署

配置host文件

| 服务     | 域名                        |
| -------- | --------------------------- |
| mysql    | mysql.codesec.asi           |
| rabbitmq | rabbitmq.asmp.asi           |
| mongodb  | mongodb.codesec.asi         |
| zipkin   | zipkin.server.asmp.asi      |
| eureka   | eureka.springcloud.asmp.asi |
| 用户中心 | asuc.asmp.asi               |
| 网关     | zuul.springcloud.asmp.asi   |

---
## Eureka部署


### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i eureka-server.tar
```

### 运行

1. 运行容器

```shell
docker run -p 8761:8761 -v /var/log/misas/eureka-server/:/home/misas/eureka-server/logs/ --net=host --name eureka-server 172.31.102.156:5000/eureka-server:latest
```

### 配置所在主机与域名关系

```
${ip} eureka.springcloud.asmp.asi
ip hostname localhost
```

---
## Zipkin安装


### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i zipkin-server.tar
```

### 运行

1. 运行容器

```shell
docker run -p 9411:9411 -v /var/log/misas/zipkin-server/:/home/misas/zipkin-server/logs/ --net=host --name zipkin-server 172.31.102.156:5000/zipkin-server:latest
```

### 配置所在主机与域名关系

```shell
${ip} zipkin.server.asmp.asi
```

---
## 代码审计后端部署

### 安装

1. 拷贝tar压缩文件
2. docker加载镜像

```shell
docker load -i business-platform.tar
```

### 运行

1. 运行容器

```shell
docker run -p 8763:8763 -v /var/log/misas/code-sec/:/home/misas/platform/logs/ --net=host --name business-platform 172.31.102.156:5000/business-platform:11.0.7 --spring.redis.host=10.236.77.32 --misas.path.resource.cloc=/home/misas/platform/resource/cloc/

```

spring.redis.host的地址改为redis.asmp.asi的地址。

---
 用户中心


使用的服务域名

| 服务   | 域名                        |
| ------ | --------------------------- |
| mysql  | mysql.asmp.asi              |
| redis  | redis.asmp.asi              |
| eureka | eureka.springcloud.asmp.asi |

---

## redis部署

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i redis.tar
```

### 运行

1. 运行容器

```shell
docker run -p 6379:6379 --restart=unless-stopped --name misas-redis -d redis:latest redis-server --appendonly yes 
```

---
## rabbitmq部署

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i rabbitmq.tar
```

### 运行

1. 运行容器

```shell
docker run -d \
  --hostname rabbit \
  --name misas-rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management
```

---
## Gateway部署

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i gateway.tar.gz
```

### 运行

1. 运行容器

```shell
docker run -v /var/log/misas/gateway/:/asuc/logs/ --net=host --name gateway misas:gateway
```

---
## 用户中心后端程序部署

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i asuc.tar.gz
```

### 运行

1. 运行容器

```shell
docker run  -v /var/log/misas/asuc/:/asuc/logs/ --net=host --name asuc misas:asuc
```



---
# 应用中台前端离线部署

| 服务 | 域名                      |
| ---- | ------------------------- |
| 网关 | zuul.springcloud.asmp.asi |

### 安装

1. 拷贝tar压缩文件，不要解压
2. docker加载镜像

```shell
docker load -i asuc-front.1.0.tar
```

### 运行

1. 运行容器

```shell
docker run --name asuc-front -ti -d --net=host --restart=always asuc-front:1.0
```

启动后访问80端口，若需要映射到其他端口，则需在命令中加入```-p 端口号:80```

![](https://img.shields.io/badge/%E4%B8%AD%E5%9B%BD%E7%94%B5%E4%BF%A1-MISAS-green)


