# MySql
在mysql配置文件加上
```ini
# 在命令头前加载当前数据
[mysql]
prompt = (\\u@\\h) [\\d]>\\
```

```sql
// 太多时可以垂直显示
Select * from tableName \G 
```

show databases;查看当前有哪几个数据库
use xxx;使用某个当前数据库
show tables;查看当前数据库有多个表

初始化数据库时，配置datadir，一般时mysql一块地方，mysql数据库放在一块地方。


https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html


mysql文件配置模板
https://github.com/jdaaaaaavid/mysql_best_configuraiton


安装mysql
脚本安装mysql binnary
5.6 升级 5.7 改软链 in-Place upgrade

mysql可以有多个配置文件
mysql --help --verbose | grep my.cnf 

多个文件，后面的会配置会替代前面的配置
show variables like '%%' 查看当前配置的参数，可以看mysql配置文件中的参数

MySQL配置参数
* 从作用域上可以划分为global和session
* 从类型上又可以划分为可修改和只读参数
* 用户可在线修改非只读参数
* 只读参数可能通过配置文件修改并重启
* 所有参数的修改都不持久化

// 查看所有连接sql的设置的变量值
use performance_schema >  select * from variables_by_thread where variable_name = ''
// 查看当前连接到数据库的线程 
show processlist;
// 5.7之后 除了一个新表 thread_id 连接号 thread_os_id 系统线程Id
select * from threads where thread_id 

用户权限管理：
GRANT

能够设置权限的文档

https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html

```sql
// 此账号可以在不同的ip上登录
create user 'david'@'%' identified by ''
// 查看某某权限
show grants for 
// 给某人test库 前面的权限
grant select,update,insert,delete on test.* to ''
// 修改密码
alter user ''@'' identified by ''
// 收回权限
revoke create,index on text.* from ''
revoke all on text.* from ''
// 将自己的权限授予给其他用户
grant select,update,insert,delete on test.* to '' with grant option

```

mysql中的四张表，表示用户的权限范围了，其中user表示全局，db库，tables_priv,collum_priv

连接MySQL实例
* 通过本地socket进行连接
* 通过TCP/IP协议远程连接
`mysql -h 192.168.6.133 -p 3306 -u root - p`
* 通过配置my.cnf免密码输入

登陆时
`mysql -uroot -p -S/*/*/mysql.sock`
如果不是，则需要在配置文件中配置相应配置可以通过 show variables like 'socket%'查看设置位置

workbench

Mysql支持SSL的连接方式

```mysql

mysql_ssl_rsa_setup，就会生成公密钥的文件
alter user xxx@'' require ssl;

```

validate_password 密码插件

配置文件、变量参数、连接、安全权限管理、多实例安装

* 一台服务器上可以安装多个MySQL实例，只要端口号，socket实例不一致就好了

mysqld_multi 

```mySql

[mysqld_multi]
mysql=/usr/local/mysql/bin/mysqld_safe
mysqladmin=/usr/local/mysql/bin/mysqladmin
log=/usr/local/mysql/mysqld_multi.log

```
## 一些常用的配置

错误日志，mysql启动过程中失败了情况
* 参数：log_err
    默认名称：机器名.err
[mysqld]
log_error =

慢查询日志
* 将运行超过某个时间阈值的SQL语句记录到文件
```ini
[mysqld]
# 开启慢查询日志
slow_query_log=1
# 慢查询日志文件的位置
slow_query_log_file=
# 慢查询的阈值，时间
long_query_time=

```

通常日志
* 记录所有的操作
不推荐开启，浪费性能

## 数据类型

**INT类型**
* TINYINT 1字节
* SMALLINT 2字节
* MEDIUMINT 3字节
* INT 4字节
* BIGINT 8字节

INT类型的属性，`UNSIGNED/SIGNED`（有无符号）、`ZEROFILL`（填充0的个数）、`AUTO_INCREMENT`（自增）。

不推荐使用UNSIGNED，范围本质上没有大的改变，UNSIGNED可能会有溢出现象发生，自增INT类型主键建议使用BIGINT。

**数字类型**
* 单精度类型：FLOAT 4字节
* 双精度类型：DOUBLE 8字节
* 高精度类型：DECIMAL（财务、账务系统必须用DECIMAL类型） 变长

FLOAT与DOUBLE这两个类型不精确，两个相同类型的相除，其结果不一定为1.

**字符串类型：**

![字符串类型](/img/type-string.png)
* BLOB=》VARBINARY
* TEXT=》VARCHAR

需要注意的：
* 在BLOB和TEXT列上创建索引时，必须指定索引前缀的长度
* VARCHAR和VARBINARY前缀的长度时可选的
* BLOB和TEXT列不能有默认值
* BLOB和TEXT列排序时只使用改列的前max_sort_length个字符

对于那些有字符集的字符串类型来说，其排序可以通过字符集的规定，默认时不区分大小写，并且默认比较到空格结束，而没有字符集的则按照十六进制来排序。

**字符串类型--ENUM&SET**
* 字符类型-集合类型
* ENUM类型最多允许65536
* SET类型最多允许64个值
* 通过sql_mode参数可以用于约束检查

```sql
CREATE TABLE t(
    user VARCHAR(30),
    sex ENUM('male','female')
)ENGINE=InnoDB;
```

**日期类型**

![日期](/img/type-date.png)

* TIMESTAMP 包含时区变化

![日期函数](/img/type-date-fun.png)

**JSO类型**

5.7版本支持Json格式数据，可以使用JSON类型替换BLOB类型。
* JSON数据有效性检查：BLOB类型无法在数据库层做到这样的约束性检查
* 查询性能的提升，查询不需要遍历所有字符串才能找到数据
* 支持部分属性索引：通过虚拟列的功能可以对JSON中的部分数据进行索引

![Json函数](/img/type-json-func.png)

## Information_schema
`Information_shcema`这个数据库中的Tables和Column表用来记录一些表和行结构信息的。

```sql

Select table_schema,table_name,engine,sys.format_bytes(data_length) as data_size FROM tables where engine <> 'InnoDB' and table_schema NoT in ('mysql','performance_schema','information_shcema');

```

## create 语法

select emp_no,salary from salaries s inner join (select emp_no empNo,max(to_date) lastDate from salaries group by emp_no) s1 on s.emp_no = s1.empNo and s.to_date = s1.lastDate;