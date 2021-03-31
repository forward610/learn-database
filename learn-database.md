| Date: 2021/3/31      |
| -------------------- |
| **Author**: Fahaxiki |

---

[TOC]

## 1数据库的神秘面纱

### 1.1初识数据库

能**保存大量数据**，通过**计算机加工而成**且能**高效访问**的数据集合称为数据库(Database简称DB)

**用来管理数据库的而计算机系统称为数据库管理系统**(Database Management System简称DBMS)

### 1.2数据库类型

| 关系型数据库       | SQL Server, Oracle, Mysql, PostgreSQL |
| ------------------ | ------------------------------------- |
| **非关系型数据库** | **MongoDB**， **Redis**， **CouchDB** |

## 2会魔法的SQL

### 2.1初识SQL

为**操作数据库**而开发的语言。国际标准化组织（ISO）为 SQL 制定了相应的标准，以此为基准的SQL 称为**标准 SQL**

#### 2.1.1SQL分类

1. 用来创建或者删除存储数据的数据库及数据库中的表等对象(数据定义语言Data Definition Language)简称**DDL**
2. 用来查询或者变更表中的记录（Data Manipulation Language,数据操纵语言）简称**DML**
3. 用来确认或者取消对数据库中的数据进行的变更。还可以对RDBMS的用户是否有权限操作数据库中的对象(数据库表等)进行设定。简称**DCL**

- DDL
    - create : 创建数据库和表等对象
    - drop  :   删除数据库和表等对象的结构
    - alter ：  修改数据库和表等对象
- DML
    - select：查询表中的数据
    - insert：向表中插入新数据
    - update：更新表中的数据
    - delete：删除表中的数据
- DCL
    - commit：变更的数据提交到数据库中
    - rollback：回滚对数据库中的数据进行变更
    - grant：赋予用户操作权限
    - revoke：取消用户的操作权限

### 2.2SQL的基本书写规则

1. 英文模式下书写SQL语句
2. 以分号结尾
3. 不区分关键字的大小写，但是插入到表中的数据是区分大小写的
4. win系统默认不区分表名及字段名的大小写
5. Linux / mac默认严格区分表名的大小写

### 2.3数据库的创建

```sql
语法：
	create database <库名>
例：
   create database shop;
```

### 2.4表的创建

```sql
语法：
	create table <表名>
    (< 列名 1> < 数据类型 > < 该列所需约束 > ,
  	 < 列名 2> < 数据类型 > < 该列所需约束 > ,);
例:
    create table product(
    	product_id char(4) not null,
        product_name varchar(100) not null,
        product_type varchar(32) not null,
        sale_price integer,
        purchase_price integer,
        regist_date date,
        primary key(product_id)
    );
```

### 2.5命名规则

- 只能使用半角英文字母、数字、下划线_ 作为数据库、表和列的名称
- 名称必须以半角英文字母开头

### 2.6数据类型的制定

- integer型

>用来指定存储整数的列的数据类型（数字型），不能存储小数

- char型

>用于存储定长字符串，当列中存储的字符串长度达不到最大长度，使用半角空格进行补足，浪费存储空间，少用

- varchar型

>用来存储定长字符串，定长字符串在字符未达到最大长度用半角空格补足，但可变长字符串不同，即使字符数未达到最大长度，也不会用半角空格补足

- date型

>用来指定存储日期（年月日）的列的数据类型（日期型）

### 2.7约束的设置

约束是除了数据类型外，对列中存储的数据进行限制追加条件的功能

`not null`是非空约束，该列必须输入数据

`primary key`是主键约束，代表该列是唯一值，可以通过该列取出特定的行的数据

### 2.8表的删除和更新

- 删除表语法

```sql
drop table <表名>;
例：
drop table product;  # 删除无法恢复，删除需谨慎！！！
```

- 添加列语法

```sql
alter table <表名> add column <列的定义>;
例：
alter table product add column product_name_pinyin varchar(100);
```

- 删除列语法

```sql
alter table <表名> drop column <列名>;
例：
alter table product drop column product_name_piyin;
```

- 清空表内容

```sql
truncate table table_name;
优点：相比drop delete,   truncate用来清除数据时，速度最快
```

- 更新数据         使用update时要注意添加where条件，否则将会把所有的行按照语句修改

```sql
语法： update <表名> set <列名> = <表达式>
--例
	update product set regist_date = '2021-03-31';  # 修改所有的注册时间
    update product set sale_price = sale_price * 10 where product_type = '厨房用具'; 
    
# update 也可以将列更新为null（俗称清空null）但是只有未设置not null约束和主键约束的才可以清空为null，其他会报错
    例： update product set regist_date = null where product_id = '0008';
```

- 多列更新

```sql
update product set sale_price = sale_price * 10 where product_type = '厨房用具';
update product set purchase_price = purchase_price / 2 where product_type = '厨房用具';
--优化代码
update product set sale_price = sale_price * 10,purchase_price = purchase_price / 2 where product_type = '厨房用具';
```

### 2.9表中插入数据

```sql
语法：  insert into <表名> (列1，列2...) values (值1，值2...);  # 默认从左到右依次插入
--优化写法
insert into product values('0005', '高压锅', '厨房用具', 6800, 5000, '2021-03-31');
```

- insert 语句中想给某一列赋予 NULL 值时，可以直接在values子句的值清单中写入 null。想要插入 null 的列一定不能设置 not null约束。

```sql
insert into product (product_id,product_name,product_type,sale_price,purchase_price,regist_date) values ('006','叉子','厨房用具',500,null,'2021-03-31');
```

- 可以向表中插入默认值（初始值）。可以通过在创建表的create table语句中设置**default**约束来设定默认值。

```sql
create table product (product_id char(4) not null, sale_price integer default 0 primary key(product_id));
```

- 可以使用insert ... select语句从其他表复制数据。

```sql
-- 例：将商品表中的数据复制到商品复制表中
insert into productocpy(product_id, product_name, product_type, sale_price, purchase_price, regist_date) select product_id, product_name, product_type, sale_price, 
purchase_price, regist_date from Product;
```

