# MySQL 5.7 安装

## 相关地址

- 官方文档：[MySQL :: MySQL Secure Deployment Guide :: 1 Introduction](https://dev.mysql.com/doc/mysql-secure-deployment-guide/5.7/en/secure-deployment-overview.html)

- 下载地址：[华为云镜像仓库](https://repo.huaweicloud.com/mysql/Downloads/MySQL-5.7/)

## 安装环境

- 操作系统：`CentOS 7`
- 用户身份:  `root`

## 安装步骤

安装目录：`/usr/local/mysql3306`

数据库版本：`5.7.xx`

以此安装为例：`mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz`

### 下载MySQL for Linux 通用二进制包

- 下载安装包

```shell
cd /usr/local && wget https://repo.huaweicloud.com/mysql/Downloads/MySQL-5.7/mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz
```

### 安装MySQL二进制包

1. 安装依赖库: `libaio`    

```shell
yum install libaio
```

2. 创建mysql用户组和用户（已有可忽略）
- 创建用户组
  
```shell
groupadd -g 27 -o -r mysql
```

- 创建用户
  
```shell
useradd -M -N -g mysql -o -r -s /bin/false -u 27 mysql
```
3. 解压安装包
   
```shell
cd /usr/local \
&& tar -zxvf mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz
```
- 创建符号链接
  
```shell
cd /usr/local \
&& ln -s mysql-5.7.33-linux-glibc2.12-x86_64 mysql3306
```
4. 添加环境变量（可选）
- 打开`.bashrc`文件

```shell
vi ~/.bashrc
```

- 文件末尾加入以下内容
  
```shell
export PATH=/usr/local/mysql3306/bin:$PATH
```

- 刷新`.bashrc`

```shell
source ~/.bashrc
```

### 安装后设置

#### 为导入和导出操作创建安全目录

```shell
cd /usr/local/mysql3306 \
&& mkdir mysql-files \
&& chown mysql:mysql mysql-files \
&& chomd 750 mysql-files
```

#### 配置服务器启动选项

```shell
mkdir /usr/local/mysql3306/etc \
&& cd /usr/local/mysql3306/etc \
&& touch my.cnf \
&& chown mysql:mysql my.cnf && chmod 644 my.cnf
```

配置`my.cnf`

```shell
vi /usr/local/mysql3306/etc/my.cnf
```

将此配置信息添加到`my.cnf`文件中：

```vim
[mysqld]
bind-address=0.0.0.0
basedir=/usr/local/mysql3306
datadir=/usr/local/mysql3306/data
socket=/tmp/mysql_3306.sock
port=3306
log-error=/usr/local/mysql3306/data/localhost.localdomain.err
user=mysql
secure_file_priv=/usr/local/mysql3306/mysql-files
local_infile=OFF
```

#### 数据库实例初始化

- 创建数据目录
  
```shell
cd /usr/local/mysql3306 && mkdir data \
&& chmod 750 data && chown mysql:mysql data
```

- 初始化数据目录

```shell
cd /usr/local/mysql3306 && bin/mysqld \
--defaults-file=/usr/local/mysql3306/etc/my.cnf \
--initialize \
--basedir=/usr/local/mysql3306 \
--datadir=/usr/local/mysql3306/data
```

- 查看初始化密码 (root@localhost: password )

```shell
head /usr/local/mysql3306/data/localhost.localdomain.err
```

```vim
[Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use 
--explicit_defaults_for_timestamp server option (see documentation for more details).
[Warning] InnoDB: New log files created, LSN=45790
[Warning] InnoDB: Creating foreign key constraint system tables.
[Warning] No existing UUID has been found, so we assume that this is the first time that this 
server has been started. Generating a new UUID: ee40ce3b-367c-11e7-adf9-080027b8b5f8.
[Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
[Warning] CA certificate ca.pem is self signed.
[Note] A temporary password is generated for root@localhost: jh;kgEza*9&t
```

#### 启动数据库服务

```shell
/usr/local/mysql3306/bin/mysqld \
--defaults-file=/usr/local/mysql3306/etc/my.cnf \
--daemonize \
--pid-file=/usr/local/mysql3306/data/mysqld.pid \ 
--basedir=/usr/local/mysql3306 \
--datadir=/usr/local/mysql3306/data
```

#### 客户端连接

- 指定`sock`文件连接，回车后输入初始化密码

```shell
/usr/local/mysql3306/bin/mysql -S /tmp/mysql_3306.sock -p
```

- 修改初始化密码

```sql
alter user root@localhost identified by '你的密码';
```

- 允许远程连接

```sql
grant all on *.* to root@'%' identified by '你的密码';
```

- 刷新授权信息

```sql
flush privileges;
```
-  **远程连接注意事项**
1. 关闭防火墙：`systemctl stop firewalld`
2. 关闭`selinux`：`setenforce 0`

#### 编写管理脚本

- 创建脚本目录

```shell
mkdir /usr/local/mysql3306/scripts
```

#### 启动脚本

- 编辑脚本

```shell
vi /usr/local/mysql3306/scripts/startup.sh
```

```bash
#!/bin/bash
/usr/local/mysql3306/bin/mysqld \
--defaults-file=/usr/local/mysql3306/etc/my.cnf \
--daemonize \
--pid-file=/usr/local/mysql3306/data/mysqld.pid \ 
--basedir=/usr/local/mysql3306 \
--datadir=/usr/local/mysql3306/data
```
-  可执行文件权限
```shell
chmod +x /usr/local/mysql3306/scripts/startup.sh
```
- 创建软链接

```shell
ln -s /usr/local/mysql3306/scripts/startup.sh \
/usr/local/mysql3306/bin/mysqld_startup_3306
```

##### 关闭脚本

- 编辑脚本

```shell
vi /usr/local/mysql3306/scripts/shutdown.sh
```

```shell
#!/bin/bash
kill -9 $(cat /usr/local/mysql3306/data/mysqld.pid)
```
-  可执行文件权限
```shell
chmod +x /usr/local/mysql3306/scripts/shutdown.sh
```
- 创建软链接

```shell
ln -s /usr/local/mysql3306/scripts/shutdown.sh \
/usr/local/mysql3306/bin/mysqld_shutdown_3306
```
> By LiRhyme
