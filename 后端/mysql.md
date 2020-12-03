制作系统服务:mysqld --install
解除系统服务:mysqld --remove
修改密码：mysqladmin -uroot -p123 password "sheng107068"
mysql已经被制作成了一个服务，可以直接去服务那去手动启动mysql
也可以通过net start（stop） MySQL启动（停止）服务，

#### 数据库相关概念

1.数据库服务器：运行数据库管理软件的计算机

2.数据库管理软件：mysql，oracle，db2，sqlserver

3.库：文件夹

4.表：文件

5.记录：事物一系列典型的特征：zs male 18

6.数据：描述事物特征的符号

#### 操作文件夹（库）

增

```mysql
create database db1 charset utf8;
```

查

```mysql
show create database db1;
show databases;
```

改：只能改字符编码

```mysql
alter database db1 charset gbk;
```

删

```mysql
drop database db1;
```

#### 操作文件（表）

​	切换文件夹：use db1;
​	查看当前所在文件夹：select database();
增:新增一个表

```mysql
create table t1(id int,name char);
```

查show

```mysql
show create table t1;查看表结构
show tables;查看这个文件夹里面的所有表
desc t1;也是查看表结构
```

改：改表的字段

```mysql
alter table t1 modify(修改的意思) name char(6)(宽度为6);改的是字段name的类型
alter table t1 change name NAME char(7); 改字段名
```

删

```mysql
drop table t1;
```

#### 操作文件内容（记录）

增

```mysql
insert t1(id,NAME) values(1,"zs"),(2,"lisi");
```

查

```mysql
select id,name from db1.t1;简单得写法：select * from db1.t1;
```

改

```mysql
update db1.t1 set name="haha";表里面的name全都改了
update db1.t1 set name="zs1" where id=1;修改某一条
```

删

```mysql
delete from t1;表里面的内容全删
delete from t1 where id=2;
```

SQL语言主要用于存取数据，查询数据，更新数据和管理关系数据库系统，SQL语言分为3种类型：
1.DOL语句，数据库定义语言：数据库，表，视图，索引，存储过程，例如CREATE DROP ALTER
2.DML语句，数据库操纵语言：插入数据INSERT，删除数据DELETE，更新数据UPDATE，查询数据SELECT
3.DCL语句，数据库控制语言：例如控制用户的访问权限GRANT，REVOKE

data文件夹：
information_schema:虚拟库，不占用磁盘空间，存储的是数据库启动后的一些参数，如用户表信息，列信息，权限信息，字符信息
performance_schema:MySQL5.5开始新增一个是数据库：主要用于收集数据库服务器性能参数，记录处理查询请求时发生的各种事件，锁等现象
mysql：授权库，主要存储系统用户的权限信息
test：MySQL数据库系统自动创建的测试数据库

1.什么是存储引擎？存储引擎就是表的类型
2.查看MySQL支持的存储引擎：show engines;
3.指定表类型/存储引擎
除了innodb类型，其他的了解
create table t1(id int)engine=innodb;
	innodb类型的表会对应两个文件，一个是t1.frm（表结构）,一个是t1.ibd（数据文件）
create table t2(id int)engine=memory;
	memory类型只有一个文件，数据是存在内存中的，不放在硬盘上，只有一个表结构
create table t3(id int)engine=blackhole;
	blackhole类型只有一个文件，是一个黑洞，数据丢进去就没了
create table t4(id int)engine=myisam;
	myisam对应三个文件，一个是t4.frm（表结构）,一个是t4.MYD（data文件），一个是t4.MYI（索引文件）

insert into t1 values(1);
insert into t2 values(1);
insert into t3 values(1);
insert into t4 values(1);

#### 表的介绍

表相当于文件，表中的一条记录就相当于文件的一行内容，不同的是，表中的一条记录有对应的标题
，成为表的字段。
create table 表名(
字段1 类型[(宽度) 约束条件],
字段2 类型[(宽度) 约束条件],
字段3 类型[(宽度) 约束条件]
);
注意：
1.在同一张表中，字段名是不能相同
2.宽度和约束条件可选
3.字段名和类型是必须的

show create table t4\G:按照一行一行显示表结构

#### 修改表结构

1.修改表名

```mysql
ALTER TABLE 表名
			RENAME 新表名；
```

2.增加字段

```mysql
ALTER TABLE 表名
			ADD 字段名 数据类型 [完整性的约束条件...],
			ADD 字段名 数据类型 [完整性的约束条件...]；
ALTER TABLE 表名
			ADD 字段名 数据类型 [完整性的约束条件...] FIRST；控制添加的字段放在哪个位置
ALTER TABLE 表名
			ADD 字段名 数据类型 [完整性的约束条件...] AFTER 字段名；把新字段添加到哪个字段后			
```

3.删除字段

```mysql
ALTER TABLE 表名
			DROP 字段名;
```

4.修改字段

```mysql
ALTER TABLE 表名
			MODIFY 字段名 数据类型 [完整性的约束条件...]
ALTER TABLE 表名
			CHANGE 旧字段名 新字段名 旧数据类型 [完整性的约束条件...]
ALTER TABLE 表名
			CHANGE 旧字段名 新字段名 新数据类型 [完整性的约束条件...]
```

#### 复制表结构

```mysql
create table t1 select host,user from mysql.user;
从mysql库中找到user表，然后将user表中的host和user表结构和内容在创建表t1的时候复制到t1这张表
```

```mysql
create table t2 select host,user from mysql.user where 1>5;where 1>5条件为假
只复制表结构，不复制内容（只拷贝想要的表结构）
create table t3 like mysql.user
只复制表结构，不复制内容(全部拷贝表结构）
```

数值类型
1.整数类型：TINYINT，SMALLINT，MEDIUMINT INT BIGINT，代表整数的范围不一样
作用：存储年龄，等级，id，各种号码等