# 1.启动服务：

~~~bash
net start mysql80
~~~

# 2.登录

~~~bash
mysql -uroot -padmin123
~~~


## 指定端口和ip登录：唯独密码-P后不能加空格

~~~bash
mysql -uroot -P3306 -h localhost -p
~~~

### 进入到mysql>下

### 显示版本：

~~~mysql
select version()
~~~


# 3.显示数据库：

~~~mysql
show databases;
~~~

# 4.创建新的数据库：

~~~mysql
create database dbtest1;
~~~


切换到要使用的数据库：

~~~mysql
use dbtest1;
~~~

创建表：

~~~mysql
create table employees(id int,name varchar(15));
~~~

显示

~~~mysql
show tables;
show create table employees;
~~~

查看表中数据：

~~~my
select * from employees;
~~~

# 5.添加数据：

~~~mysq
insert into employees values(1001,'Tom');
~~~

如果中文报错，按照下面步骤修改：
# 6.查看编码命令：

~~~mysql
show variables like 'character_%';
show variables like 'collation_%';
~~~

打开my.ini
在[mysql]下添加

~~~markdown
default-character-set=utf8
~~~


在[mysqld]下添加

~~~markdown
character-set-server=utf8
collation-server=utf8_general_ci #此处操作存在问题，5.7版本无需修改就可以正常输入中文
~~~



