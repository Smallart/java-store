# 安装
* [资料](https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html)

安装过程

```shell
# mysql的安装要依赖于libaio包，所以按如下方式安装
apt-get install libaio1
# 添加mysql的用户组
groupadd mysql
# 在用户组中创建一个不需要登录的mysql用户
useradd -r -g mysql -s /bin/false mysql
# 未本路径下的mysql文件夹与mysql-8.0.22-el7-x86_64（这个是安装文件目录）创建一个软链 也就是可以在mysql文件夹中访问到连接文件夹中的内容，这个与inode有关
ls -s mysql-8.0.22-el7-x86_64/ mysql
# 创建mysql-files文件夹
mkdir mysql-files
# 修改文件夹的所有者和分组
chown mysql:mysql mysql-files
# 设置所有人、所有组和其他的对应权限，所拥有人的权限为rwx，而所有组的则是rx
chmod 750 mysql-files
# 创建mysql_log文件夹
mkdir mysql_log
# 创建mysql_log文件夹，修改其拥有人和分组，这个用来存储
chown -R mysql:mysql mysql_log
# 这个是mysql的配置文件，用于对mysql数据库变量进行一些设置
vi /ect/my.cnf
#########################################
[client]

[mysqld]
# 这个是mysql的安装位置
basedir=/usr/local/mysql
# mysql数据库文件存放位置
datadir=/usr/local/mysql-files/data
# mysql错误日志
log-error=/usr/local/mysql_log/mysql.err

#########################################
# 初始化mysql的初始数据库（5.7）的初始胡方式，此后会在文件中生成一个用户root的随机密码
sudo  /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql-files/data
#
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
# 创建全局的软链，这样就可以在bash的任何文件夹下使用mysql命令
ln -s /usr/local/mysql/bin/mysql /usr/bin
# 开启mysql服务
systemctl start mysqld
# 登录mysql
mysql -u root -p

```

?> 当使用`systemctl start mysqld`可能会出现如下错误 `Failed to start mysqld.service: Unit mysqld.service not found.` 解决方式如下

```shell
sudo systemctl enable mysqld.service
sudo systemctl status mysqld
● mysqld.service - LSB: start and stop MySQL
   Loaded: loaded (/etc/init.d/mysqld; bad; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)
 sudo /etc/init.d/mysqld start
```

## mysql的一些配置

```ini
# my.cnf配置文件
# 显示当前所在的数据库和选用的表
[mysql]
prompt = (\\u@\\h) [\\d]>\\

```
**查看当前mysql的各个参数的配置的值**
```mysql
show variables like '%%';

set variablesname = '';

```

要注意的是：MySQL配置参数
* 从作用域上可以划分为global和session
* 从类型上又可以划分为可修改和只读参数
* 用户可在线修改非只读参数
* 只读参数可能通过配置文件修改并重启
* 所有参数的修改都不持久化

![mysql参数](/img/mysql-variables.png)

## 多实例安装

## 权限

![权限种类](/img/common-privilege.png)

```sql
# 查看权限
show grants for u*;

# 赋予全部权限，一般由管理员操作
grant all on *:* to u*:*


# 将自己的权限授予给其他用户
grant select,update,insert,delete on test.* to '' with grant option

# 回收权限
revoke all on text.* from ''

```

这个权限的修改，实际上是修改了`mysql`数据库中的user（全局），db（数据库），tables_priv（表）,collum_priv（行）的这四张表。

