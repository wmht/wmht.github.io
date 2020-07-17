---
title: 部署 Cloudera Manager
tags:
  - 'CDH'
  - 'Cloudera Manager'
date: 2020-07-17 15:39:09
---

### 教程

[Cloudera Manager 官网安装步骤](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/configure_cm_repo.html)

[Cloudera Manager 官网下载网页](https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cm_6_version_download.html#concept_dcs_x11_3jb)

[Cloudera Manager 6.3.1 yum源](https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/)

[Cloudera Manager 6.3.1 离线包](https://archive.cloudera.com/cm6/6.3.1/repo-as-tarball/)

[CDH 官网下载网页](https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_cdh_63_download.html#cdh_632-download)

[CDH 6.3.2 yum源](https://archive.cloudera.com/cdh6/6.3.2/redhat7/yum/)

[CDH 6.3.2 离线包](https://archive.cloudera.com/cdh6/6.3.2/parcels/)

[离线部署参考](https://blog.csdn.net/cloudmq/article/details/100876717)



### 离线包

```
vim /etc/yum.repos.d/cloudera-manager.repo 

[cloudera-manager]
name=Cloudera Manager
baseurl=https://wmqe-yum-repository.oss-cn-zhangjiakou-internal.aliyuncs.com/cm6/6.3.1/
gpgkey =https://wmqe-yum-repository.oss-cn-zhangjiakou-internal.aliyuncs.com/cm6/6.3.1/RPM-GPG-KEY-cloudera
gpgcheck=0
```



### 驱动

```shell
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar zxvf mysql-connector-java-5.1.46.tar.gz
mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```



### 创建数据库

```mysql
# scm
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'wmq20151118';

# amon
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'wmq20151118';

# rman
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'wmq20151118';

# hue
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; 
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'wmq20151118';

# hive
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'wmq20151118';

# sentry
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;   
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'wmq20151118';

# nav
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;      
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'wmq20151118';

# navms
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'wmq20151118';

# oozie
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'wmq20151118';

# flush
FLUSH PRIVILEGES;
```

执行脚本

```shell
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm
```



```shell
# 下载allkeys.asc文件，存在仓库
https://archive.cloudera.com/cm6/6.3.1/allkeys.asc
```



### 启动

```shell
systemctl start cloudera-scm-server
```



```shell
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
```



### Oozie ExtJS library

```shell
# 安装oozie主机上执行
cd /opt/cloudera/parcels/CDH/lib/oozie/libext/
wget http://archive.cloudera.com/gplextras/misc/ext-2.2.zip
unzip ext-2.2.zip
chown oozie:oozie -R ext-2.2
```

