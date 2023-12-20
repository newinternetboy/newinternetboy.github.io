---
title:  "安装卸载mysql(ubuntu)"
date:   2023-12-20 15:43:26 +0800
categories: mysql
---
# 安装
mysql  Ver 8.0.35

[官方文档](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)
1. 更新apt
```bash
sudo apt-get update
```

2. 安装mysql
```bash
sudo apt-get install mysql-server
```
注意安装完后，登录账户及密码在/etc/mysql/debian.cnf文件中
登陆之后，设置root账户密码及权限
```bash
mysql> use mysql;
mysql> alter user 'root'@'localhost'identified with mysql_native_password by '$password';
```

3. 操作mysql服务
* 启动
```bash
$> systemctl start mysql
```
* 停止
```bash
$> systemctl stop mysql
```
* 状态
```bash
$> systemctl status mysql
```
* 重启
```bash
$> systemctl restart mysql
```


# 卸载
1. 关闭mysql服务
```bash
$> sudo systemctl stop mysql
```

2. 删除各种client
```bash
$> sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
```

3. 删除配置文件及日志
```bash
$> sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql
```

4. 移除额外的包
```bash
$> sudo apt autoremove
```

5. 清除磁盘空间
```bash
$> sudo apt autoclean
```