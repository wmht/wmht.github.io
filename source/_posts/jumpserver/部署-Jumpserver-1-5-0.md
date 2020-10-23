---
title: 部署 Jumpserver 1.5.0
tags:
  - jumpserver
categories:
  - jumpserver
abbrlink: 13359
date: 2020-10-23 16:37:14
---

## 部署 Jumpserver 1.5.0

### 1、基础设置

\# 版本说明

```bash
# 操作系统：centos7.6
# jumpserver：1.5.0
```

\# 升级所有包同时也升级软件和系统内核

```bash
yum update -y
```

\# selinux配置

```bash
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

\# 安装依赖包

```bash
yum -y install wget gcc epel-release git vim
```

 

### 2、安装Redis

\# 安装 Redis, Jumpserver 使用 Redis 做 cache 和 celery broke

```bash
yum -y install redis
systemctl enable redis
systemctl start redis
```

 

### 3、安装MySQL

\# 安装 MySQL（centos7下叫mariadb）

```bash
yum -y install mariadb mariadb-devel mariadb-server MariaDB-shared
systemctl enable mariadb
systemctl start mariadb
```

\# 创建数据库 Jumpserver 并授权

```bash
mysql -uroot
create database jumpserver default charset 'utf8';
grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by 'pawd123';
flush privileges;
```

 

### 4、配置Python环境

\# 安装 Python3.6

```bash
yum -y install python36 python36-devel
```

\# 配置并载入 Python3 虚拟环境

```bash
cd /opt
python3.6 -m venv py3
source /opt/py3/bin/activate
```

\# 看到下面的提示符代表成功, 以后运行 Jumpserver 都要先运行以上 source 命令, 载入环境后默认以下所有命令均在该虚拟环境中运行

```bash
(py3) [root@localhost py3]
```

 

### 5、安装Jumpserver

\# 下载 Jumpserver

```bash
cd /opt/
git clone --depth=1 https://github.com/jumpserver/jumpserver.git
```

\# 安装依赖 RPM 包

```bash
yum -y install $(cat /opt/jumpserver/requirements/rpm_requirements.txt)
```

\# 安装 Python 库依赖

```bash
pip install --upgrade pip setuptools
pip install -r /opt/jumpserver/requirements/requirements.txt
```

\# 复制 Jumpserver 配置文件

```bash
cd /opt/jumpserver
cp config_example.yml config.yml
```

\# 生成随机SECRET_KEY

加密秘钥 生产环境中请修改为随机字符串, 请勿外泄, PS: 纯数字不可以

```bash
SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`
echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc
```

\# 生成随机BOOTSTRAP_TOKEN

预共享Token coco和guacamole用来注册服务账号, 不在使用原来的注册接受机制

```bash
BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16` 
echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc
```

\# 查看SECRET_KEY、BOOTSTRAP_TOKEN

```bash
cat ~/.bashrc
SECRET_KEY=c1NVKkBQonnam9CqX8AGmJTr3DYFnam9CqX8A
BOOTSTRAP_TOKEN=nam9CqX8A
```

\# 修改配置文件

```bash
sed -i "s/SECRET_KEY:/SECRET_KEY: $SECRET_KEY/g" /opt/jumpserver/config.yml
sed -i "s/BOOTSTRAP_TOKEN:/BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN/g" /opt/jumpserver/config.yml
sed -i "s/# DEBUG: true/DEBUG: false/g" /opt/jumpserver/config.yml
sed -i "s/# LOG_LEVEL: DEBUG/LOG_LEVEL: ERROR/g" /opt/jumpserver/config.yml
sed -i "s/# SESSION_EXPIRE_AT_BROWSER_CLOSE: false/SESSION_EXPIRE_AT_BROWSER_CLOSE: true/g" /opt/jumpserver/config.yml
sed -i "s/DB_PASSWORD: /DB_PASSWORD: $DB_PASSWORD/g" /opt/jumpserver/config.yml
```

\# 查看完整 config.yml 配置文件

cat /opt/jumpserver/config.yml

```bash
# SECURITY WARNING: keep the secret key used in production secret!
# 加密秘钥 生产环境中请修改为随机字符串，请勿外泄, 可使用命令生成 
# $ cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 49;echo
SECRET_KEY: c1NVKkBQonnam9CqX8AGmJTr3DYFnam9CqX8A

# SECURITY WARNING: keep the bootstrap token used in production secret!
# 预共享Token coco和guacamole用来注册服务账号，不在使用原来的注册接受机制
BOOTSTRAP_TOKEN: nam9CqX8A

# Development env open this, when error occur display the full process track, Production disable it
# DEBUG 模式 开启DEBUG后遇到错误时可以看到更多日志
DEBUG: false

# DEBUG, INFO, WARNING, ERROR, CRITICAL can set. See https://docs.djangoproject.com/en/1.10/topics/logging/
# 日志级别
LOG_LEVEL: ERROR
# LOG_DIR: 

# Session expiration setting, Default 24 hour, Also set expired on on browser close
# 浏览器Session过期时间，默认24小时, 也可以设置浏览器关闭则过期
# SESSION_COOKIE_AGE: 86400
SESSION_EXPIRE_AT_BROWSER_CLOSE: true

# Database setting, Support sqlite3, mysql, postgres ....
# 数据库设置
# See https://docs.djangoproject.com/en/1.10/ref/settings/#databases

# SQLite setting:
# 使用单文件sqlite数据库
# DB_ENGINE: sqlite3
# DB_NAME: 

# MySQL or postgres setting like:
# 使用Mysql作为数据库
DB_ENGINE: mysql
DB_HOST: 127.0.0.1
DB_PORT: 3306
DB_USER: jumpserver
DB_PASSWORD: pawd123
DB_NAME: jumpserver

# When Django start it will bind this host and port
# ./manage.py runserver 127.0.0.1:8080
# 运行时绑定端口
HTTP_BIND_HOST: 0.0.0.0
HTTP_LISTEN_PORT: 8080

# Use Redis as broker for celery and web socket
# Redis配置
REDIS_HOST: 127.0.0.1
REDIS_PORT: 6379
# REDIS_PASSWORD: 
# REDIS_DB_CELERY: 3
# REDIS_DB_CACHE: 4

# Use OpenID authorization
# 使用OpenID 来进行认证设置
# BASE_SITE_URL: http://localhost:8080
# AUTH_OPENID: false  # True or False
# AUTH_OPENID_SERVER_URL: https://openid-auth-server.com/
# AUTH_OPENID_REALM_NAME: realm-name
# AUTH_OPENID_CLIENT_ID: client-id
# AUTH_OPENID_CLIENT_SECRET: client-secret
#
# Use Radius authorization
# 使用Radius来认证
# AUTH_RADIUS: false
# RADIUS_SERVER: localhost
# RADIUS_PORT: 1812
# RADIUS_SECRET: 


# OTP settings
# OTP/MFA 配置
# OTP_VALID_WINDOW: 0
# OTP_ISSUER_NAME: Jumpserver
```

\# 运行 Jumpserver

```bash
cd /opt/jumpserver
./jms start all -d
```

\# 开机自启

```bash
vim /usr/lib/systemd/system/jms.service

[Unit]
Description=jms
After=network.target mariadb.service redis.service
Wants=mariadb.service redis.service

[Service]
Type=forking
Environment="PATH=/opt/py3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"
ExecStart=/opt/jumpserver/jms start all -d
ExecReload=
ExecStop=/opt/jumpserver/jms stop

[Install]
WantedBy=multi-user.target
```

 

### 6、安装coco

```bash
cd /opt/
git clone https://github.com/jumpserver/coco.git
cd /opt/coco
yum -y install $(cat /opt/coco/requirements/rpm_requirements.txt)
pip install -r /opt/coco/requirements/requirements.txt
```

\# 复制配置文件

```bash
cp config_example.yml config.yml
```

\# 配置文件完整配置

cat /opt/coco/config.yml

```bash
# 项目名称, 会用来向Jumpserver注册, 识别而已, 不能重复
# NAME: {{ Hostname }}

# Jumpserver项目的url, api请求注册会使用
CORE_HOST: http://127.0.0.1:8080

# Bootstrap Token, 预共享秘钥, 用来注册coco使用的service account和terminal
# 请和jumpserver 配置文件中保持一致，注册完成后可以删除
BOOTSTRAP_TOKEN: nam9CqX8A

# 启动时绑定的ip, 默认 0.0.0.0
# BIND_HOST: 0.0.0.0

# 监听的SSH端口号, 默认2222
# SSHD_PORT: 2222

# 监听的HTTP/WS端口号，默认5000
# HTTPD_PORT: 5000

# 项目使用的ACCESS KEY, 默认会注册,并保存到 ACCESS_KEY_STORE中,
# 如果有需求, 可以写到配置文件中, 格式 access_key_id:access_key_secret
# ACCESS_KEY: null

# ACCESS KEY 保存的地址, 默认注册后会保存到该文件中
# ACCESS_KEY_FILE: data/keys/.access_key

# 加密密钥
# SECRET_KEY: null

# 设置日志级别 [DEBUG, INFO, WARN, ERROR, FATAL, CRITICAL]
LOG_LEVEL: ERROR

# 日志存放的目录
# LOG_DIR: logs

# SSH白名单
# ALLOW_SSH_USER: all

# SSH黑名单, 如果用户同时在白名单和黑名单，黑名单优先生效
# BLOCK_SSH_USER:
#   -

# 和Jumpserver 保持心跳时间间隔
# HEARTBEAT_INTERVAL: 5

# Admin的名字，出问题会提示给用户
# ADMINS: ''

# SSH连接超时时间 (default 15 seconds)
# SSH_TIMEOUT: 15

# 语言 [en,zh]
# LANGUAGE_CODE: zh

# SFTP的根目录, 可选 /tmp, Home其他自定义目录
# SFTP_ROOT: /tmp

# SFTP是否显示隐藏文件
# SFTP_SHOW_HIDDEN_FILE: false

# 是否复用和用户后端资产已建立的连接(用户不会复用其他用户的连接)
# REUSE_CONNECTION: true
```

\# 后台启动coco

```bash
/opt/coco/cocod start -d
```

\# 开机自启

```bash
vim /usr/lib/systemd/system/coco.service

[Unit]
Description=coco
After=network.target jms.service

[Service]
Type=forking
PIDFile=/opt/coco/coco.pid
Environment="PATH=/opt/py3/bin"
ExecStart=/opt/coco/cocod start -d
ExecReload=
ExecStop=/opt/coco/cocod stop

[Install]
WantedBy=multi-user.target
```

 

### 7、下载Luna

\# 安装 Web Terminal 前端: Luna 需要 Nginx 来运行访问 访问(https://github.com/jumpserver/luna/releases)下载对应版本的 release 包, 直接解压, 不需要编译

```bash
cd /opt
wget https://github.com/jumpserver/luna/releases/download/1.5.0/luna.tar.gz
```

\# 如果网络有问题导致下载无法完成可以使用下面地址

```bash
wget https://demo.jumpserver.org/download/luna/1.5.0/luna.tar.gz
```

\# 解压

```bash
tar xf luna.tar.gz
chown -R root:root luna
```

 

### 8、安装Nginx

\# 安装 Nginx, 用作代理服务器整合 Jumpserver 与各个组件

```bash
vim /etc/yum.repos.d/nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
yum -y install nginx
```

\# 配置 Nginx 整合各组件

```bash
rm -rf /etc/nginx/conf.d/default.conf
vim /etc/nginx/conf.d/jumpserver.conf

server {
    # http自动跳转到https
    listen       80;
    server_name jump.wmht.com;    
    rewrite ^ https://$http_host$request_uri? permanent;       
}

server {
    # 代理端口, 以后将通过此端口进行访问, 不再通过8080端口
    listen 443 ssl;

    ssl_certificate ssl/jump.wmht.com.pem;
    ssl_certificate_key ssl/jump.wmht.com.key;

    server_name jump.wmht.com;  # 修改成你的域名或者注释掉

    client_max_body_size 100m;  # 录像及文件上传大小限制

    location /luna/ {
        try_files $uri / /index.html;
        alias /opt/luna/;  # luna 路径, 如果修改安装目录, 此处需要修改
    }

    location /media/ {
        add_header Content-Encoding gzip;
        root /opt/jumpserver/data/;  # 录像位置, 如果修改安装目录, 此处需要修改
    }

    location /static/ {
        root /opt/jumpserver/data/;  # 静态资源, 如果修改安装目录, 此处需要修改
    }

    location /socket.io/ {
        proxy_pass       http://localhost:5000/socket.io/;  # 如果coco安装在别的服务器, 请填写它的ip
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /coco/ {
        proxy_pass       http://localhost:5000/coco/;  # 如果coco安装在别的服务器, 请填写它的ip
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location /guacamole/ {
        proxy_pass       http://localhost:8081/;  # 如果guacamole安装在别的服务器, 请填写它的ip
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

    location / {
        proxy_pass http://localhost:8080;  # 如果jumpserver安装在别的服务器, 请填写它的ip
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

\# 运行 Nginx

```bash
nginx -t
systemctl enable nginx
systemctl start nginx
```

 

### 9、访问

\# 访问 UI (注意 没有 :8080 通过 nginx 代理端口进行访问)：

http://192.168.244.144
默认账号: admin 密码: admin 到会话管理-终端管理 接受 coco 等应用的注册

\# 测试ssh连接
ssh -p2222 admin@192.168.244.144
密码: admin

 

### 10、附启动命令

```bash
#启动
systemctl start mariadb
systemctl start redis
systemctl start jms
systemctl start coco
systemctl start nginx
```

 