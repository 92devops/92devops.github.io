---
title: 通用二进制安装 MySQL5.7
date: 2023-11-29 23:31:06
categories:
- MySQL
tags:
- mysql
archive:
- MySQL
---

1、安装依赖

```
apt-get install libaio1
```

2、获取软件包

```
wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
```

3、创建mysql User 和 Group

```
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```


4、解压程序包

```
mkdir -pv  /data/apps/
tar -xf /root/mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz  -C /data/apps/
```

5、创建软链

```
cd /data/apps/
ln -sv mysql-5.7.9-linux-glibc2.5-x86_64 mysql
```

6、准备数据目录

```
mkdir /data/apps/mysql/{data,logs}
```

7、初始化数据目录

```
cd /data/apps/mysql
mkdir mysql-files
chown mysql:mysql mysql-files data logs
chmod 750 mysql-files
```
```
bin/mysqld --initialize --user=mysql --basedir=/data/apps/mysql --datadir=/data/apps/mysql/data 
  ......
2023-04-11T02:43:44.783949Z 1 [Note] A temporary password is generated for root@localhost: bR8oqd*Qwdd!

bin/mysql_ssl_rsa_setup --datadir=/data/apps/mysql/data
```

8、提供相关文件

```
# cp support-files/my-default.cnf  /etc/my.cnf
cat  > /etc/my.cnf <<EOF
[client]
port = 3306
socket = /data/apps/mysql/data/mysql.sock

[mysqld]
character_set_server=utf8mb4
skip_name_resolve=1
innodb_file_per_table=1
max_connections=20000
basedir = /data/apps/mysql/
datadir = /data/apps/mysql/data
port = 3306
socket = /data/apps/mysql/data/mysql.sock
log-error=/data/apps/mysql/logs/mysqld.log
pid-file=/data/apps/mysql/data/mysqld.pid
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
EOF

cp support-files/mysql.server /etc/init.d/mysqld
```

9、启动服务

```
/etc/init.d/mysqld start 
```

10、连接数据库报错解决

```
mysql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory

ln -sv /usr/lib/x86_64-linux-gnu/libncurses.so.6.2 /usr/lib/x86_64-linux-gnu/libncurses.so.5
```

11、修改密码

```
mysql -uroot -p'bR8oqd*Qwdd!' -S /data/apps/mysql/data/mysql.sock 
alter user 'root'@'localhost' identified by 'root123';
flush privileges;
```
```

mysql -uroot -p'root123' -S /data/apps/mysql/data/mysql.sock  -e 'show databases;'
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```
