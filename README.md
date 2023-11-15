# allNote
all mistake note

111
https://blog.csdn.net/weixin_44446230/article/details/123710202
# 1.docker安装mysql 8.0/5.7版本，执行sql报错sql_mode=only_full_group_by问题解决
Cause: java.sql.SQLSyntaxErrorException: Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column ‘wechatworkx.u.user_name’ which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
报错原因
MySql从5.7版本开始默认开启only_full_group_by规则，
规则核心原则如下，没有遵循原则的sql会被认为是不合法的sql

不允许select后面的非聚合列不出现在group by中
不允许name的存在
示例sql
```sql
SELECT
   id,
   name
FROM
    table_name
GROUP BY
    id
```
方案1.执行sql解决，但重启mysql服务会失效，
下面可以作为mysql8版本教程，主要区别是sql_mode中没有NO_AUTO_CREATE_USER
查看当前状态
```sql
SELECT @@GLOBAL.sql_mode;
SELECT @@SESSION.sql_mode;

```

查看结果，主要是ONLY_FULL_GROUP_BY影响，需要将这个删除，
```sql
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

```
执行sql修改
```sql
set @@GLOBAL.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
set @@SESSION.sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

```
方案2.通过将本地mysql配置文件，挂载到docker服务中，重启服务还是生效，需要删除容器重新运行，mysql数据会丢失，注意备份数据
1.查看mysql服务名
````sql
 docker ps -a
````
2. 进入到docker，mysql服务中，mysql-test为服务名，需要根据实际情况替换
```
docker exec -it mysql-test bash

```
3.查看msyql的配置文件，获取读取配置文件夹位置
```shell
cat /etc/mysql/my.cnf

```
在liunx服务器上创建mysqld.cnf配置文件
注意这里如果是在win电脑上编辑，然后上传到liunx服务器，见下面的mysqld.cnf无权限，读取失败，注意点进行修改文件权限

```shell
# Copyright (c) 2014, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock

# datadir 设置MySQL数据库的数据的存放目录
datadir         = /var/lib/mysql


#log-error      = /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address   = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# 修改mysql加密插件，mysql8.0.11 默认值为caching_sha2_password，密码复杂可能验证不过
default_authentication_plugin=mysql_native_password

#如果要更改docker mysql的端口，请先修改mysql.cnf文件中的端口
port=3306 

#从 show variables like '%sql_mode'; 里面复制出来，并去掉了 only_full_group_by
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

```

7.删除mysql容器
mysql所有数据都会丢失，注意备份数据

```shell
docker stop mysql-test
docker rm mysql-test

```
8.启动命令增加将liunx配置文件映射到docker，mysql文件夹中。
-v 挂载文件
/home/docker/mysql/mysqld.cnf liunx服务器文件路径
:/etc/mysql/conf.d/mysqld.cnf 挂载到docker，mysql中文件路径

```shell
docker run -itd -p 3306:3306 --name mysql-test  -v /home/docker/mysql/mysqld.cnf:/etc/mysql/conf.d/mysqld.cnf   -e MYSQL_ROOT_PASSWORD=123456 mysql

```
9.执行sql查看当前状态，没有ONLY_FULL_GROUP_BY表示成功
```shell
SELECT @@GLOBAL.sql_mode;

```
方案3.直接进入进入到mysql容器中，新建mysqld.cnf配置文件，好处是直接重启容易生效，不用备份数据，但是删除容器之后需要再次操作
执行方案2中第1,2,3,5步
需要在docker中的mysql执行上传或编辑，需要安装相关依赖
```shell
上传文件提示rz、sz命令找不到或不成功：
执行命令：apt-get update && apt-get install lrzsz
vi命令找不到
执行命令：apt-get update && apt-get install vim

```
然后将win电脑编辑的文件上传到docker中mysql文件夹中
修改文件权限
```shell
chmod 644 mysqld.cnf 

```
停止mysql之后在启动mysql
```shell
docker stop mysql-test
docker start mysql-test

```
异常错误
发生错误，具体看日志，查看日志中error和自己操作步骤相关的关键字，然后网上进行搜索相关教程
使用命令查看日志
```shell
docker logs -f mysql-test

```
mysqld.cnf无权限，读取失败
查考教程
https://blog.csdn.net/exeron/article/details/120982051

mysqld: File ‘/etc/mysql/conf.d/mysqld.cnf’ not found (OS errno 13 - Permission denied)
mysqld: [ERROR] Stopped processing the ‘includedir’ directive in file /etc/mysql/my.cnf at line 29.






