| DATE：2021-04-03     |
| -------------------- |
| **Author**：Fahaxiki |

---

[TOC]

## 3.1 视图

- **视图**是一个虚拟表，不同于直接操作数据表，视图是依据selct语句来创建的，在这张虚拟表上做SQL操作

### 3.1.1 为什么会存在视图

1. 通过定义视图可以将频繁使用的select语句保存以提高效率
2. 通过定义视图可以使用户看到的数据更加清晰
3. 通过定义视图可以不对外公开数据表全部字段，增强数据的保密性
4. 通过定义视图可以降低数据的冗余

### 3.1.2 如何创建视图

```sql
--语法
create view <视图名称>(<列名>,<列名2>,...) as <select语句>；
/* select语句需写在 as 关键字之后，select语句中列的排列顺序和视图中列的排列顺序相同，select中的1列就是视图中的1列，依次类推。注意视图名在数据库中是唯一的，不能与其他视图和表重名
```

- 注意事项

一般`DBMS`中定义视图时不能使用order by 语句

```sql
--错误示例
create view productsum (product_type,cnt_product) as select product_type,count(*) from product group by product_type order by product_type;
```

为什么不能使用order by子句呐？ 因为视图和表一样，**数据行都是没有顺序的**

*在 MySQL中视图的定义是允许使用 ORDER BY 语句的，但是若从特定视图进行选择，而该视图使用了自己的 ORDER BY 语句，则视图定义中的 ORDER BY 将被忽略。*

- 基于单表的视图

```sql
create view productsum(product_type,cnt_product) as select product_type,count(*) from product group by product_type;
```

```sql
--多表视图
create table shop_product (
    shop_id char(4) not null,shop _name varchar(200) not null,
	product_id char(4) not null,quanity integer not null,primary key (shop_id,product_id));
insert into shop_product (shop_id,shop_name,product_id,quantity) values ('00A','京东','0001',30);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000A',	'东京',		'0002',	50);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000A',	'东京',		'0003',	15);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000B',	'名古屋',	'0002',	30);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000B',	'名古屋',	'0003',	120);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000B',	'名古屋',	'0004',	20);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000B',	'名古屋',	'0006',	10);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000B',	'名古屋',	'0007',	40);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000C',	'大阪',		'0003',	20);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000C',	'大阪',		'0004',	50);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000C',	'大阪',		'0006',	90);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000C',	'大阪',		'0007',	70);
INSERT INTO shop_product (shop_id, shop_name, product_id, quantity) VALUES ('000D',	'福冈',		'0001',	100);
```

在product和shop_product表基础上创建视图

```sql
create view view_shop_product (product_type,sale_price,shop_name) as select product_type,sale_price,shop_name from product,shop_product where product_id = shop_product.product_id;
```

```sql
--查询
select sale_price,shop_name from view_shop_product where product_type = '衣服';
```

### 3.1.3如何修改视图结构

```sql
--语法
alter view <视图名> as <select语句>
```

- 修改视图

```sql
--示例
alter view productSum as select product_type,sale_price from Product where regist_date > '2009-09-11';
```

### 3.1.4 如何更新视图内容

- 对于一个视图来说，如果包含以下结构的任意一种都是不可以被更新的：
    - 聚合函数sum()、min()、max()、count()、
    - distinct 关键字
    - group by 子句
    - having 子句
    - union 或 union all 运算符
    - from 子句中包含多个表

```sql
--更新视图
update productSum set sale_price = '5000' where product_type = '办公用品';
此时观察原表也可以发现数据也被更新了
```

**注意：这里虽然修改成功了，但是并不推荐这种使用方式。而且我们在创建视图时也尽量使用限制不允许通过视图来修改表**

### 3.1.5 如何删除视图

drop view <视图名1> [,<视图名2>...]  注意：需要有相应的权限才能成功删除

- 删除视图

```sql
drop view productSum;
```

## 3.2 子查询

```sql
--示例
select stu_name from 
(select stu_name,count(*) as stu_cnt from students_info group by stu_age) as studentSum;
```

### 3.2.1 什么是子查询

子查询指一个查询语句嵌套在另一个查询语句内部的查询，这个特性从MySQL4.1开始引入，在select子句中先计算子查询，子查询结果作为外层另一个查询的过滤条件，拆线呢可以基于一个表或者多个表

- **子查询和视图的关系**

子查询就是将用来定义视图的 SELECT 语句直接用于 FROM 子句当中。其中AS studentSum可以看作是子查询的名称，而且由于子查询是一次性的，所以子查询不会像视图那样保存在存储介质中， 而是在 SELECT 语句执行之后就消失了。

### 3.2.2嵌套子查询

```sql
select product_type, cnt_product
from (select *
        from (select product_type, 
                      count(*) as cnt_product
                from product 
               group by product_type) as productsum
       where cnt_product = 4) as productsum2;
```

**虽然嵌套子查询可以查询出结果，但是随着子查询嵌套的层数的叠加，SQL语句不仅会难以理解而且执行效率也会很差，所以要尽量避免这样的使用**

### 3.2.3 标量子查询

**标量**：就是单一的意思      **单一**：就是要执行SQL语句仅返回一个值，即表中具体的**某一行的某一列**

```sql
--查询出销售单价高于平均销售单价的商品
--查询出注册日期最晚的那个商品
select product_id,product_name,sale_price from product where sale_price > (select avg(sale_price) from product);
--使用标量子查询
select product_id,product_name sale_price,(select avg(sale_price) from product) as avg_price from product;
```

### 3.2.4 关联子查询

```sql
select product_type,product_name,sale_price from product as p1 
where sale_price > (select avg(sale_price) from product as p2 where p1.product_type = p2.product_type group by product_type);
```

- 关联子查询与子查询的联系

```sql
--查询出销售单价高于平均销售单价的商品
select product_id,product_name,sale_price from product 
where sale_price > (select avg(sale_price) from product);
--选取出各商品种类中高于该商品种类的平均销售单价的商品
select product_type,product_name,sale_price from product as p1
where sale_price > (select avg(sale_price) from product as p2 where p1.product_type = p2.product_type group by product_type);
```

- 关联查询执行过程

    1. 首先执行不带where的主查询
    2. 根据主查询讯结果匹配product_type，获取子查询结果
    3. 将子查询结果再与主查询结合执行完整的SQL语句

    *在子查询中像标量子查询，嵌套子查询或者关联子查询可以看作是子查询的一种操作方式即可。*

## 3.3 各种各样的函数

- 算术函数 （用来进行数值计算的函数）
- 字符串函数 （用来进行字符串操作的函数）
- 日期函数 （用来进行日期操作的函数）
- 转换函数 （用来转换数据类型和值的函数）
- 聚合函数 （用来进行数据聚合的函数）

### 3.3.1 算数函数

- ABS — 绝对值     		语法：`ABS(数值)`

    ABS 函数用于计算一个数字的绝对值，表示一个数到原点的距离。当ABS函数的参数为`NULL`时，返回值也是`NULL`

- MOD—求余数              语法：`MOD(被除数，除数)`

    MOD是计算除法余数（求余的函数）时module的缩写。小数没有余数的概念，只能对整数列求余数。

    注意：主流的 DBMS 都支持 MOD 函数，只有SQL Server 不支持该函数，其使用`%`符号来计算余数。

- Round—四舍五入        语法：`Round(对象数值，保留小数的位数)`

    Round函数用来进行四舍五入操作

    注意：当参数 **保留小数的位数** 为变量时，可能会遇到错误，请谨慎使用变量。

    ```sql
    select m,ABS(m) as abs_col, n,p, mod(n,p) as mod_col, round(m,1) as round_cols from samplemath;
    ```

    ### 3.3.2 字符串函数

    ```sql
    --创建samplestr
    use shop;
    drop table if exists samplestr;
    create table samplestr (str1 varchar(40),str2 varchar(40),str3 varchar(40));
    --插入数据
    START TRANSACTION;
    INSERT INTO samplestr (str1, str2, str3) VALUES ('opx',	'rt', NULL);
    INSERT INTO samplestr (str1, str2, str3) VALUES ('abc', 'def', NULL);
    INSERT INTO samplestr (str1, str2, str3) VALUES ('太阳',	'月亮', '火星');
    INSERT INTO samplestr (str1, str2, str3) VALUES ('aaa',	NULL, NULL);
    INSERT INTO samplestr (str1, str2, str3) VALUES (NULL, 'xyz', NULL);
    commit;
    ```

    - concat —  拼接          语法：`concat(str1,str2,str3)`

        MySQL中使用concat函数进行拼接

    - length —  字符串长度        语法：`length(字符串)`

    - lower — 小写转换

        lower函数只能针对英文字母使用，它会将参数中的而字符串全都转换为小写。不适用英文以外的场合，不影响原本就是小写的字符。类似的，upper函数用于大写转换。

    - replace — 字符串的替换      语法：`replace(对象字符串，替换前的字符串，替换后的字符串)`

    - substring —  字符串的截取   语法：`substring(对象字符串from截取的起始位置for街区的字符串)`

        ```sql
        -- SUBSTRING 函数可以截取出字符串中的一部分字符串。截取的起始位置从字符串最左侧开始计算，索引值起始为1
        select str1,str2,str3,concat(str1,str2,str3) as str_concat,length(str1) as len_str,lower(str1) as low_str,replace(str1,str2,str3) as rep_str,substring(str1 from 3 for 2) as sub_str from samplestr;
        ```

        - substring_index  字符串按索引截取

        语法： `substring_index(原始字符串，分隔符，n)`

        该函数用来获取原始字符串按照分割符分割后，第n个分割符之前或之后的字符串，支持正向和反向索引，索引起始值分别为1和-1

        ```sql
        select substring_index('www.mysql.com','.',2);
        +----------------------------------------+
        | substring_index('www.mysql.com','.',2) |
        +----------------------------------------+
        | www.mysql                              |
        +----------------------------------------+
        1 row in set (0.00 sec)
        select substring_index('www.mysql.com','.',-2);
        +-----------------------------------------+
        | substring_index('www.mysql.com','.',-2) |
        +-----------------------------------------+
        | mysql.com                               |
        +-----------------------------------------+
        1 row in set (0.01 sec)
        ```

        获取第一个元素比较容易，获取第二个元素/第n个元素可以采用二次拆分的写法

        ```sql
        select substring_index('www.mysql.com','.',1);
        +----------------------------------------+
        | substring_index('www.mysql.com','.',2) |
        +----------------------------------------+
        | www                              |
        +----------------------------------------+
        1 row in set (0.00 sec)
        select substring_index(substring_index('www.mysql.com','.',2),'.',-1);
        +----------------------------------------+
        | substring_index('www.mysql.com','.',2) |
        +----------------------------------------+
        | mysql                              |
        +----------------------------------------+
        1 row in set (0.00 sec)
        ```

### 3.3.3 日期函数

- current_date —— 获取当前日期

```sql
select current_date;
+--------------+
| current_date |
+--------------+
| 2021-04-05   |
+--------------+
1 row in set (0.01 sec)
```

- current_timestamp —— 当日期和时间

```sql
select current_timestamp;
+---------------------+
| current_timestamp   |
+---------------------+
| 2021-04-05 10:22:13 |
+---------------------+
1 row in set (0.00 sec)
```

- extract —— 截取日期元素     语法：`extract(日期元素 from 日期)`

    使用 EXTRACT 函数可以截取出日期数据中的一部分，例如“年”“月”，或者“小时”“秒”等。该函数的返回值并不是日期类型而是数值类型

    ```sql
    select current_timestamp as now,
    extract(year from current_timestamp) as year,
    extract(month from current_timestamp) as month,
    extract(day from current_timestamp) as day,
    extract(hour from current_timestamp) as hour,
    extract(minute from current_timestamp) as minute,
    extract(second from current_timestamp) as second;
    ```

### 3.3.4 转换函数

- cast —— 类型转换      语法：`cast(转换前的值 as 想要转换的数据类型)`

    ```sql
    --将字符串类型转换为数值类型
    select cast('001' as signed integer) as int_col;
    +---------+
    | int_col |
    +---------+
    |       1 |
    +---------+
    1 row in set (0.00 sec)
    -- 将字符串类型转换为日期类型
    select cast('2009-12-14' as date) as date_col;
    +------------+
    | date_col   |
    +------------+
    | 2009-12-14 |
    +------------+
    1 row in set (0.00 sec)
    ```

- coalesce —— 将null转换为其他值      语法：`coalesce(数据1，数据2，数据3...)`

    返回可变参数a中左侧开始第一个不是null的值。参数个数是可变的，淫才可以根据需要无限增加。

    ```sql
    -- 在SQL语句中将null转换为其他值时就会用到转换函数
    select coalesce(null,11) as col_1,
    coalesce(null,'hello word',null) as col_2,coalesce(null,null,'2020-11-01') as col_3;
    +-------+------------+------------+
    | col_1 | col_2      | col_3      |
    +-------+------------+------------+
    |    11 | hello word | 2020-11-01 |
    +-------+------------+------------+
    1 row in set (0.00 sec)
    ```

## 3.4  谓词

### 3.4.1 什么是谓词

谓词就是返回值为真值的函数。包括`true  / false  / unknown`

- like
- between
- is null、is not null
- in
- exists

### 3.4.2  like谓词   用于字符串的部分一致查询

```sql
--创建表
create table samplelike (strcol varchar(6) not null,primary key(strcol));
--插入数据
START TRANSACTION; -- 开始事务
INSERT INTO samplelike (strcol) VALUES ('abcddd');
INSERT INTO samplelike (strcol) VALUES ('dddabc');
INSERT INTO samplelike (strcol) VALUES ('abdddc');
INSERT INTO samplelike (strcol) VALUES ('abcdd');
INSERT INTO samplelike (strcol) VALUES ('ddabc');
INSERT INTO samplelike (strcol) VALUES ('abddc');
COMMIT; -- 提交事务
```

- 前方一致：选取出'dddabc'

    ```sql
    select * from samplelike where strcol like 'ddd%';
    其中%是代表零个或多个任意字符串的特殊符号
    ```

- 中间一致：选取出“abcddd”,“dddabc”,“abdddc”

    ```sql
    select * from samplelike where strcol like '%ddd%';
    ```

- 后方一致：选取出“abcddd“

    ```sql
    select * from samplelike where strcol like '%ddd';
    ```

- `_`下划线匹配任意1个字符

    ```sql
    select * from samplelike where strcol like 'abc_';
    +--------+
    | strcol |
    +--------+
    | abcdd  |
    +--------+
    1 row in set (0.00 sec)
    ```

### 3.4.3    between谓词       用于范围查询

```sql
-- 选取销售单价为100~1000元的商品
select product_name,sale_price from product where sale_price between 100 and 1000;
-- 不包含临界值
select prodct_name,sale_price from product where sale_price > 100 and sale_price <1000;
```

### 3.4.4   is null  、is not null      用于判断是否为null

```sql
-- 选取某写值为null的数据，不可以使用=
select product_name,purchase_price from product where purchase_price is null;
--
select product_name,purchase_price from product where purchase_price is not null;
```

### 3.4.5    in谓词            or的简便用法

```sql
-- 通过or指定多个进货单假进行查询
select product_name,purchase_price from product where purchase_price = 320 or purchase_price = 500
or purchase_price = 5000;

--IN谓词来替换上述的SQL语句
select product_name,purchase_price from product where purcharse_price in(320,500,5000);
+--------------+----------------+
| product_name | purchase_price |
+--------------+----------------+
| T恤衫        |            500 |
| 打孔器       |            320 |
| 高压锅       |           5000 |
+--------------+----------------+
3 rows in set (0.01 sec)
-- 查询进货单价不是320、500、5000的
select product_name,purchase_price from product where purchase_price not in (320,500,5000);
## 注意：使用 in 和 not in 时是无法选取出NULL数据的 
```

### 3.4.6  使用子查询作为IN为谓词的参数

- IN谓词和NOT IN谓词  **可以使用子查询作为参数**   可以说**能将表作为IN的参数**，还**可以说将试图作为IN的参数**

```sql
drop table if exists shopproudct;
create table shopproduct (shop_id char(4) not null,shop_name,varchar(200) not null,product_id char(4) not null,quantity integer not null,primary key (shop_id,product_id));
--插入数据
start transaction;
INSERT INTO shopproduct (shop_id, shop_name, product_id, quantity) VALUES ('000A', '东京', '0001', 30);
INSERT INTO shopproduct (shop_id, shop_name, product_id, quantity) VALUES ('000A', '东京', '0002', 50);
INSERT INTO shopproduct (shop_id, shop_name, product_id, quantity) VALUES ('000A', '东京', '0003', 15);
INSERT INTO shopproduct (shop_id, shop_name, product_id, quantity) VALUES ('000B', '名古屋', '0002', 30);
INSERT INTO shopproduct (shop_id, shop_name, product_id, quantity) VALUES ('000B', '名古屋', '0003', 120);
COMMIT;
```

假设我么需要取出大阪在售商品的销售单价，该如何实现呢？

```sql
-- 第一步，取出大阪门店的在售商品 `product_id ;
select product_id from shopproduct where shop_id = '000C';
-- 第二步，取出大阪门店在售商品的销售单价 `sale_price
select product_id from shopproduct where product_id in (select product_id from shopproduct where shop_id = '000C');

-- 子查询展开后的结果
select product_name,sale_price from product where product_id in ('0003','0004','0006','0007');
```

1. 使用in谓词就需要经常更新sql语句，效率低，提高了维护成本
2. 使用子查询可保持sql语句不变，极大提高了程序的可维护性

- NOT  IN也支持子查询

    ```sql
    -- NOT IN 使用子查询作为参数，取出未在大阪门店销售的商品的销售单价
    select product_name,sale_price from product where product_id not in (select product_id from shopproduct where shop_id = '000A');
    ```

### 3.4.7   EXIST 谓词

- 作用：**“判断是否存在满足某种条件的记录”**

    ```sql
    --使用 EXIST 选取出大阪门店在售商品的销售单价
    select product_name,sale_price from product as p where exists (
    select * from shopproduct as sp where sp.shop_id='000C' and sp.product_id = p.product_id);
    ```

- exist的参数

    ```sql
    --exist 在右侧书写1个参数，通常都会是一个子查询
    (select * from shopproduct as sp where sp.shop_id = '000C' and sp.product_id = p.product_id)
    ```

- 子查询中的select *

    ```sql
    select product_name,sale_price from product as p where exists (select 1 from shopproduct as sp where sp.shop_id = '000C' and sp.product_id = p.product_id);
    +--------------+------------+
    | product_name | sale_price |
    +--------------+------------+
    | 运动T恤      |       4000 |
    | 菜刀         |       3000 |
    | 叉子         |        500 |
    | 擦菜板       |        880 |
    +--------------+------------+
    4 rows in set (0.00 sec)
    -- 使用NOT ECIST 替换NOT IN
    select product_name,slae_price from product as p where not exists (select * from shopproduct as sp where sp.shop_id = '000A' and sp.product_id = p.product_id);
    /* NOT EXIST 与 EXIST 相反，当“不存在”满足子查询中指定条件的记录时返回真（TRUE）。
    ```

    ## 3.5    CASE 表达式

    - CASE 表达式是函数的一种、语法分为简单CASE表达式和搜索CASE表达式两种，但搜索CASE表达式包含简单CASE表达式的全部功能

        ```sql
        case when <求值表达式> then <表达式> <求值表达式> then <表达式> ... else <表达式> end
        /* 依次判断 when 表达式是否为真值，是则执行 THEN 后的语句，如果所有的 when 表达式均为假，则执行 ELSE 后的语句。
        无论多么庞大的 CASE 表达式，最后也只会返回一个值
        ```

    - 用法

        因为表中的记录并不包含“A ： ”或者“B ： ”这样的字符串，所以需要在 SQL 中进行添加。并将“A ： ”“B ： ”“C ： ”与记录结合起来

        ```sql
        -- 假设 实现如下结果
        A: 衣服  B: 办公用品 C: 厨房用具
        -- 应用场景1：根据不同分支得到不同列值----
        select product_name,
            case when product_type = '衣服' then concat('A: ',product_type)
                 when product_type = '办公用品' then concat('B: ',product_type)
                 when product_type = '厨房用具' then concat('C: ',product_type)
                 else null -- ELSE 子句也可以省略不写
        	end as abc_product_type
        from product;
        -- 应用场景2：实现列方向上的聚合-----
        select product_type,sum(sale_price) as sum_price from product group by product_type;
        假如要在列的方向上展示不同种类额聚合值，该如何写呢？
        --聚合函数 + CASE WHEN 表达式即可实现该效果
        --对按照商品种类计算出的销售单价合计值进行行列转换
        select sum(case when product_type = '衣服' then sale_price else 0 end) as sum_price_clothes,
        	   sum(case when product_type='厨房用具' then sale_price else 0 end) as sum_price_kitchen,
        	   sum(case when product_type='办公用品' then sale_price else 0 end) as sum_price_office
        from product;
        --扩展内容：应用场景3：实现行转列-----
        -- CASE WHEN 实现数字列 score 行转列
        select name,
         	   sum(case when subject = '语文' then sale_price else null end) as chinese,
        	   sum(case when subject ='数学' then sale_price else null end) as math,
        	   sum(case when subject ='外语' then sale_price else null end) as english
        from score group by name;
        ```

        - **当待转换列为数字时，可以使用**`SUM AVG MAX MIN`**等聚合函数；**
        - **当待转换列为文本时，可以使用**`MAX MIN`**等聚合函数**



