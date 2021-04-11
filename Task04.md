| DATE：2021-04-06     |
| -------------------- |
| **Author**：Fahaxiki |

---

[TOC]

## 4.1 表的加减法

### 4.1.1 集合运算

在标准 SQL 中, 分别对检索结果使用 `UNION`, `INTERSECT,` `EXCEPT` 来将检索结果进行并,交和差运算, 像`UNION`,`INTERSECT`, `EXCEPT`这种用来进行集合运算的运算符称为**集合运算符**

### 4.1.2 表的加法——UNION

```sql
select product_id,product_name from product union select product_id,product_name from product2;
--UNION 等集合运算符通常都会除去重复的记录.
```

### 4.1.3 UNION 与 OR谓词

union 在同表查询效率方面高

分别使用 UNION 或者 OR 谓词,找出毛利率不足 30%或毛利率未知的商品.

```sql
--使用or谓词
select * from product where sale_price / purchase_price <1.3 or sale_price/purchase_price is null;
-- 使用union
select * from product where sale_price / purchase_price <1.3 union select * from product where sale_price /purchase_price is null;
```

### 4.1.4包含重复行的集合运算UNION ALL

商店决定对product表中利润低于50%和售价低于1000的商品提价, 请使用UNION ALL 语句将分别满足上述两个条件的结果取并集. 查询结果类似下表

```sql
select * from product where sale_price < 1000 union all 
select * from product where sale_price > 1.5 * purchase_price;
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
| 0006       | 叉子         | 厨房用具     |        500 |           NULL | 2009-09-20  |
| 0007       | 擦菜板       | 厨房用具     |        880 |            790 | 2008-04-28  |
| 0008       | 圆珠笔       | 办公用品     |        100 |           NULL | 2009-11-11  |
| 0001       | T恤衫        | 衣服         |       1000 |            500 | 2009-09-20  |
| 0002       | 打孔器       | 办公用品     |        500 |            320 | 2009-09-11  |
+------------+--------------+--------------+------------+----------------+-------------+
6 rows in set (0.00 sec)
```

- bag模型与set模型

    **1.该元素是否至少在一个 bag 里出现过, 2.该元素在两个 bag 中的最大出现次数** 

### 4.1.5隐式类型转换

通常来说, 我们会把类型完全一致, 并且代表相同属性的列使用 UNION 合并到一起显示, 但有时候, 即使数据类型不完全相同, 也会通过隐式类型转换来将两个类型不同的列放在一列里显示, 例如字符串和数值类型:

```sql
select product_id,product_name,'1' from product union select product_id,product_name,sale_price from product2;
```

- 使用 SYSDATE()函数可以返回当前日期时间, 是一个日期时间类型的数据, 试测试该数据类型和数值,字符串等类型的兼容性.

    ```sql
    select sysdate(),sysdate(),sysdate() union select 'chars',123,null;
    --结果如下
    +---------------------+---------------------+---------------------+
    | sysdate()           | sysdate()           | sysdate()           |
    +---------------------+---------------------+---------------------+
    | 2021-04-07 17:41:46 | 2021-04-07 17:41:46 | 2021-04-07 17:41:46 |
    | chars               | 123                 | NULL                |
    +---------------------+---------------------+---------------------+
    2 rows in set (0.00 sec)
    ```

### 4.1.6 mysql8.0不支持交运算intersect

集合的交, 就是两个集合的公共部分, 由于集合元素的互异性, 集合的交只需通过文氏图就可以很直观地看到它的意义.

### 4.1.7差集，补集与表的减法

集合A和B做减法只是将集合A中也同时属于集合B的元素减掉。

- mysql8.0不支持except运算

```sql
--找出只存在于product表但不存在于product2表的商品.
-- 使用in子句的实现方法
select * from product where product_id not in (select product_id from product2);
mysql> select * from product where product_id not in (select product_id from product2);
Empty set (0.00 sec)
```

```sql
--except与not谓词
--使用NOT谓词进行集合的减法运算, 求出product表中, 售价高于2000,但利润低于30%的商品, 结果应该如下表所示.
select * from product where sale_price > 2000 and product_id not in (select product_id from product where sale_price < 1.3 * purchase_price);
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0003       | 运动T恤      | 衣服         |       4000 |           2800 | NULL        |
| 0005       | 高压锅       | 厨房用具     |       6800 |           5000 | 2009-01-15  |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

### 4.1.8对称差

```sql
--使用product表和product2表的对称差来查询哪些商品只在其中一张表
--提示使用not in实现俩个表的差集
select * from product where product_id not in (select product_id from product2) union select * from product2 where product_id not in (select product_id from product);
+------------+--------------+--------------+------------+----------------+-------------+
| product_id | product_name | product_type | sale_price | purchase_price | regist_date |
+------------+--------------+--------------+------------+----------------+-------------+
| 0009       | 手套         | 办公用品     |        800 |            640 | 2009-11-11  |
| 0010       | 水壶         | 厨房用具     |       2000 |           1600 | 2009-10-11  |
+------------+--------------+--------------+------------+----------------+-------------+
2 rows in set (0.00 sec)
```

## 4.2连接（join）

### 4.2.1内连接（inner    join）

```sql
--内连接
from <tb_1> inner join <tb_2> on <condition(s)>
--示例
select sp.shop_id,sp.shop_name,p.product_id,p.product_name,p.sale_price,sp.quantity from shopproduct as sp inner join product as p on sp.product_id = p.product_id;
```

- 进行连接时需要在from子句中使用多张表 

    ```sql
    from shopproduct as sp inner join product as p
    ```

- 必须使用on子句来指定连接条件

- select子句中的列最好按照表名，列名的格式来使用

### 4.2.2 给和where字句使用内连接

```sql
--第一种增加 WEHRE 子句的方式, 就是把上述查询作为子查询, 用括号封装起来, 然后在外层查询增加筛选条件
select * from (select sp.shop_id,sp.shop_name,p.product_id,p.product_name,p.product_type,p.sale_price,sp.quantity from shopproduct as sp inner join product as p on sp.product_id = p.product_id) as step1 where shop_name = '东京' and product_type = '衣服';
+---------+-----------+------------+--------------+--------------+------------+----------+
| shop_id | shop_name | product_id | product_name | product_type | sale_price | quantity |
+---------+-----------+------------+--------------+--------------+------------+----------+
| 000A    | 东京      | 0001       | T恤衫        | 衣服         |       1000 |       30 |
| 000A    | 东京      | 0003       | 运动T恤      | 衣服         |       4000 |       15 |
+---------+-----------+------------+--------------+--------------+------------+----------+
2 rows in set (0.01 sec)
--WHERE 子句将在 FROM 子句之后执行, 也就是说, 在做完 INNER JOIN … ON 得到一个新表后, 才会执行 WHERE 子句, 那么就得到标准的写法:
select sp.shop_id,sp.shop_name,sp.product_id,p.product_id,p.product_name,p.product_type,p.sale_price,sp.quantity from shopproduct as sp inner join product as p on sp.product_id = p.product_id where sp.shop_name = '东京' and p.product_type = '衣服';
--执行顺序
select----> where ----> select
--还可以将 WHERE 子句中的条件直接添加在 ON 子句中, 这时候 ON 子句后最好用括号将连结条件和筛选条件括起来.
select sp.shop_id,sp.shop_name,sp.product_id,p.product_id,p.product_name,p.product_type,p.sale_price,sp.quantity from shopproduct as sp inner join product as p on (sp.product_id = p.product_id and sp.shop_name = '东京' and p.product_type = '衣服');
```

```sql
--在结合 WHERE 子句使用内连结的时候, 我们也可以更改任务顺序, 并采用任务分解的方法,先分别在两个表使用 WHERE 进行筛选,然后把上述两个子查询连结起来.
select sp.shop_id,sp.shop_name,sp.product_id,p.product_id,p.product_name,p.product_type,p.sale_price,sp.quantity from (select * from shopproduct where shop_name = '东京') as sp inner join (select * from product where product_type = '衣服') as p on sp.product_id = p.product_id;
```

![](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210411130251560.png)

```sql
-- 不使用子查询
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.product_type,p.purchase_price from shopproduct as sp inner join product as p on sp.product_id = p.product_id where p.product_type='衣服';
--使用子查询
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.purchase_price from shopproduct as sp inner join (select product_id,product_name,product_type,purchase_price from product where product_type = '衣服') as p on sp.product_id = p.product_id;
```

![](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210411131445324.png)

```sql
--不使用子查询
select sp.shop_id,sp.shop_name,sp.product_id,sp.quantity,p.product_id,p.product_name,p.product_type,p.sale_price from shopproduct as sp inner join product as p on sp.product_id = p.product_id where sp.shop_id = '000A' and p.sale_price <= 2000;
--使用子查询
select sp.shop_id,sp.shop_name,sp.quantity,sp.product_id,p.product_id,p.product_name,p.product_type,p.sale_price from (select shop_id,shop_name,product_id,quantity from shopproduct where shop_id = '000A') as sp inner join (select product_id,product_name,product_type,sale_price from product where sale_price <= 2000) as p on sp.product_id = p.product_id;
+---------+-----------+----------+------------+------------+--------------+--------------+------------+
| shop_id | shop_name | quantity | product_id | product_id | product_name | product_type | sale_price |
+---------+-----------+----------+------------+------------+--------------+--------------+------------+
| 000A    | 东京      |       30 | 0001       | 0001       | T恤衫        | 衣服         |       1000 |
| 000A    | 东京      |       50 | 0002       | 0002       | 打孔器       | 办公用品     |        500 |
+---------+-----------+----------+------------+------------+--------------+--------------+------------+
2 rows in set (0.00 sec)
```

### 4.2.3 结合group by 子句使用内连接

如果**分组列**和**被聚合的列不在同一张表**, 且二者都未被用于连结两张表, 则只能**先连结, 再聚合**.

```sql
select sp.shop_id,sp.shop_name,max(p.sale_price) as max_price from shopproduct as sp inner join product as p on sp.product_id = p.product_id group by sp.product_id,sp.shop_name;
```

### 4.2.4内连接与关联子查询

```sql
select product_type,product_name,sale_price from product as p1 where sale_price > (select avg(sale_price) from product as p2 where p1.product_type = p2.product_type group by product_type);
--首先, 使用 GROUP BY 按商品类别分类计算每类商品的平均价格
select product_type,avg(sale_price) as avg_price from product group by product_type;
--接下来, 将上述查询与表 product 按照 product_type (商品种类)进行内连结
select p1.product_id,p1.product_name,p1.product_type,p1.sale_price,p2.avg_price from product as p1 inner join (select product_type,avg(sale_price) as avg_price from product group by product_type) as p2 on p1.product_type = p2.product_type;
--最后, 增加 WHERE 子句, 找出那些售价高于该类商品平均价格的商品
select p1.product_id,p1.product_name,p1.product_type,p1.sale_price,p2.avg_price from product as p1 inner join (select product_type,avg(sale_price) as avg_price from product group by product_type) as p2 on p1.product_type = p2.product_type where p1.sale_price > p2.avg_price;
+------------+--------------+--------------+------------+-----------+
| product_id | product_name | product_type | sale_price | avg_price |
+------------+--------------+--------------+------------+-----------+
| 0002       | 打孔器       | 办公用品     |        500 |  300.0000 |
| 0003       | 运动T恤      | 衣服         |       4000 | 2500.0000 |
| 0004       | 菜刀         | 厨房用具     |       3000 | 2795.0000 |
| 0005       | 高压锅       | 厨房用具     |       6800 | 2795.0000 |
+------------+--------------+--------------+------------+-----------+
4 rows in set (0.00 sec)
--代码对比
SELECT  P1.product_id
       ,P1.product_name
       ,P1.product_type
       ,P1.sale_price
       ,AVG(P2.sale_price) AS avg_price
  FROM product AS P1
 INNER JOIN product AS P2
    ON P1.product_type=P2.product_type
 WHERE P1.sale_price > P2.sale_price
 GROUP BY P1.product_id,P1.product_name,P1.product_type,P1.sale_price,P2.product_type;
```

### 4.2.5自然连接

自然连结并不是区别于内连结和外连结的第三种连结, 它其实是内连结的一种特例–当两个表进行自然连结时, 会按照两个表中都包含的列名来进行等值内连结, 此时无需使用 ON 来指定连接条件.

```sql
select * from shopproduct natural join product;
--写出自然连结等价的内连结.
select sp.product_id,sp.shop_id,sp.shop_name,sp.quantity,p.product_name,p.product_type,p.sale_price,p.purchase_price,p.regist_date from shopproduct as sp inner join product as p on sp.product_id = p.product_id;
--求表 product 和表 product2 中的公共部分, 也可以用自然连结来实现:
select * from product natural join product2;
```

```sql
select * from (select product_id,product_name from product) as a natural join (select product_id,product_name from product2) as b;
```

### 4.2.6使用连接交集

```sql
--使用内连结求product 表和product2 表的交集.
select p1.* from product as p1 inner join product2 as p2 on (p1.product_id = p2.prodcut_id and p1.product_name = p2.product_name and p1.product_type = p2.product_type and p1.sale_price = p2.sale_price and p1.regist_date = p2.regist_date);
```

## 4.3 外连接

左连结会保存左表中无法按照 ON 子句匹配到的行, 此时对应右表的行均为缺失值; 右连结则会保存右表中无法按照 ON 子句匹配到的行, 此时对应左表的行均为缺失值; 而全外连结则会同时保存两个表中无法按照 ON子句匹配到的行, 相应的另一张表中的行用缺失值填充.

```sql
--左连接
from <tb_1> left outer join <tb_2> on <condition(s)>
--右连接
from <tb_1> right outer join <tb_2> on <condition(s)>
--全外连接
from <tb_1> full out join <tb_2> on <condition(s)>
```

### 4.3.1左连接、右连接

由于连结时可以交换左表和右表的位置, 因此左连结和右连结并没有本质区别.接下来我们先以左连结为例进行学习. 所有的内容在调换两个表的前后位置, 并将左连结改为右连结之后, 都能得到相同的结果. 稍后再介绍全外连结的概念.

### 4.3.2使用左连结从两个表获取信息

```sql
--统计每种商品分别在哪些商店有售, 需要包括那些在每个商店都没货的商品.
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.sale_price from product as p left outer join shopproduct as sp on sp.product_id = p.product_id;
```

- 要点1：选取出单张表中全部的信息
- 要点2：使用left、right 来指定主表

### 4.3.3结合where子句使用左连接

```sql
--使用外连结从shopproduct表和product表中找出那些在某个商店库存少于50的商品及对应的商店,注意高压锅和圆珠笔两种商品在所有商店都无货, 所以也应该包括在内.
select p.product_id,p.product_name,p.sale_price,sp.shop_id,sp.shop_name,sp.quantity from product as p left outer join shopproduct as sp on sp.product_id = p.product_id where quantity < 50;
-- SQL查询的执行顺序(FROM->WHERE->SELECT),我们发现那些主表中无法被匹配到的行就被WHERE条件筛选掉了
mysql> select p.product_id,p.product_name,p.sale_price,sp.shop_id,sp.shop_name,sp.quantity from product as p left outer join (select * from shopproduct where quantity < 50) as sp on sp.product_id = p.product_id;
+------------+--------------+------------+---------+-----------+----------+
| product_id | product_name | sale_price | shop_id | shop_name | quantity |
+------------+--------------+------------+---------+-----------+----------+
| 0001       | T恤衫        |       1000 | 000A    | 东京      |       30 |
| 0003       | 运动T恤      |       4000 | 000A    | 东京      |       15 |
| 0002       | 打孔器       |        500 | 000B    | 名古屋    |       30 |
| 0004       | 菜刀         |       3000 | 000B    | 名古屋    |       20 |
| 0006       | 叉子         |        500 | 000B    | 名古屋    |       10 |
| 0007       | 擦菜板       |        880 | 000B    | 名古屋    |       40 |
| 0003       | 运动T恤      |       4000 | 000C    | 大阪      |       20 |
| 0005       | 高压锅       |       6800 | NULL    | NULL      |     NULL |
| 0008       | 圆珠笔       |        100 | NULL    | NULL      |     NULL |
+------------+--------------+------------+---------+-----------+----------+
9 rows in set (0.00 sec)
```

## 4.4多表连接

遗憾的是, MySQL8.0 目前还不支持全外连结, 不过我们可以对左连结和右连结的结果进行 UNION 来实现全外连结

```sql
--创建表
create table Inventoryproduct (inventory_id char(4) not null,product_id char(4) not null,inventory_quantity integer not null,primary key(inventory_id,product_id));
----- DML：插入数据
START TRANSACTION;
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0001', 0);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0002', 120);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0003', 200);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0004', 3);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0005', 0);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0006', 99);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0007', 999);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P001', '0008', 200);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0001', 10);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0002', 25);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0003', 34);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0004', 19);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0005', 99);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0006', 0);
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0007', 0 );
INSERT INTO Inventoryproduct (inventory_id, product_id, inventory_quantity)
VALUES ('P002', '0008', 18);
COMMIT;
-- 根据上表及shopproduct 表和product 表, 使用内连接找出每个商店都有那些商品, 每种商品的库存总量分别是多少
select sp.shop_id,sp.shop_name,p.product_name,p.sale_price,ip.inventory_quantity from shopproduct as sp inner join product as p on sp.product_id = p.product_id inner join Inventoryproduct as ip on sp.product_id = ip.product_id where ip.inventory_id = 'P001';
```

```sql
--外连结一般能比内连结有更多的行, 从而能够比内连结给出更多关于主表的信息, 多表连结的时候使用外连结也有同样的作用.
select p.product_id,p.product_name,p.sale_price,sp.shop_id,sp.shop_name,ip.inventory_quantity from product as p left outer join shopproduct as sp on sp.product_id = p.product_id left outer join Inventoryproduct as ip on sp.product_id = ip.product_id;
```

### 4.4.1 ON子句进阶——非等值连接

除了使用相等判断的等值连结, 也可以使用比较运算符来进行连接. 实际上, 包括比较运算符(<,<=,>,>=, BETWEEN)和谓词运算(LIKE, IN, NOT 等等)在内的所有的逻辑运算都可以放在 ON 子句内作为连结条件

- 非等值自左结实现排名

```sql
-- 希望对 product 表中的商品按照售价赋予排名. 一个从集合论出发,使用自左连结的思路是, 对每一种商品,找出售价不低于它的所有商品, 然后对售价不低于它的商品使用 COUNT 函数计数. 例如, 对于价格最高的商品
select product_id,product_name,sale_price,count(p2_id) as rank_id from (select p1.product_id,p1.product_name,p1.sale_price,p2.product_id as p2_id,p2.product_name as p2_name,p2.sale_price as p2_price from product as p1 left outer join product as p2 on p1.sale_price <= p2.sale_price) as x group by product_id,product_name,sale_price order by rank_id;
+------------+--------------+------------+---------+
| product_id | product_name | sale_price | rank_id |
+------------+--------------+------------+---------+
| 0005       | 高压锅       |       6800 |       1 |
| 0003       | 运动T恤      |       4000 |       2 |
| 0004       | 菜刀         |       3000 |       3 |
| 0001       | T恤衫        |       1000 |       4 |
| 0007       | 擦菜板       |        880 |       5 |
| 0002       | 打孔器       |        500 |       7 |
| 0006       | 叉子         |        500 |       7 |
| 0008       | 圆珠笔       |        100 |       8 |
+------------+--------------+------------+---------+
8 rows in set (0.01 sec)
```

1. count函数的参数是列名，会忽略该列中的缺失值，参数为 * 时则不忽略缺失值

2. 果两个商品的价格相等, 则会导致两个商品的排名错误, 例如, 叉子和打孔器的排名应该都是第六, 但上述查询导致二者排名都是第七. 试修改上述查询使得二者的排名均为第六.

3.  进行排名有专门的函数, 这是 MySQL 8.0 新增加的窗口函数中的一种(窗口函数将在下一章学习), 但在较低版本的 MySQL 中只能使用上述自左连结的思路.

    使用非等值自左连结进行累计求和:

![](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210411192451562.png)

```sql
--要找出满足:1.比该商品售价更低的, 或者是 2.该种商品自身,以及 3.如果 A 和 B 两种商品售价相等,则建立连结时, 如果 P1.A 和 P2.A,P2.B 建立了连接, 则 P1.B 不再和 P2.A 建立连结, 因此根据上述约束条件, 利用 ID 的有序性, 进一步将上述查询改写为
select product_id,product_name,sale_price,sum(p2_price) as cum_price from (select p1.product_id,p1.product_name,p1.sale_price,p2.product_id as p2_id,p2.product_name as p2_name,p2.sale_price as p2_price from product as p1 left outer join product as p2 on ((p1.sale_price > p2.sale_price) or (p1.sale_price = p2.sale_price and p1.product_id <= p2.product_id)) order by p1.sale_price,p1.product_id) as x group by product_id,product_name,sale_price order by sale_price,cum_price;
```

### 4.4.2交叉连接——cross join（笛卡尔积）

两个集合做**笛卡尔积**, 就是使用集合 A 中的每一个元素与集合 B 中的每一个元素组成一个有序的组合.

```sql
--1.使用关键字 CROSS JOIN 显式地进行交叉连结
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.sale_price from shopproduct as sp cross join product as p;
--使用逗号分隔两个表,并省略 ON 子句
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.sale_price from shopproduct as sp,product as p;
```

- 连接与笛卡尔积的关系

- ```sql
    select sp.*,p.* from shopproduct as sp cross join product as p;
    --笛卡尔乘积增加筛选条件 SP.product_id=P.product_id, 就得到了和内连结一致的结果
    SELECT SP.*, P.*
      FROM shopproduct AS SP 
     CROSS JOIN product AS P
     WHERE SP.product_id = P.product_id;
    ```

- 连接的特定语法和过时语法

```sql
select sp.shop_id,sp.shop_name,sp.product_id,p.product_name,p.sale_price from shopproduct as sp cross join product as p where sp.product_id = p.product_id;
```



## 习题

### 4.1找出 product 和 product2 中售价高于 500 的商品的基本信息

```sql
select * from product where sale_price>500 union select * from product2 where sale_price>500;
```

### 4.2借助对称差的实现方式, 求product和product2的交集

```sql
select * from (select * from product union select * from product2) product where product_id not in (select product_id from (select * from product where product_id not in (select product_id from product2) union select * from product2 where product_id not in (select product_id from product)))product2;
```

### 4.3每类商品中售价最高的商品都在哪些商店有售 ？

```sql
select p1.product_id,p1.product_name,p1.product_type,mp.max_price,p2.shop_id,p2.shop_name from product p1 inner join (select product_type,max(sale_price) as max_price from product group by product_type) mp on p1.product_type = mp.product_type and p1.sale_price = mp.max_price left join shop_product p2 on p2.product_id = p1.product_id order by product_id;
+------------+--------------+--------------+-----------+---------+-----------+
| product_id | product_name | product_type | max_price | shop_id | shop_name |
+------------+--------------+--------------+-----------+---------+-----------+
| 0002       | 打孔器       | 办公用品     |       500 | 000A    | 东京      |
| 0002       | 打孔器       | 办公用品     |       500 | 000B    | 名古屋    |
| 0003       | 运动T恤      | 衣服         |      4000 | 000A    | 东京      |
| 0003       | 运动T恤      | 衣服         |      4000 | 000B    | 名古屋    |
| 0003       | 运动T恤      | 衣服         |      4000 | 000C    | 大阪      |
| 0005       | 高压锅       | 厨房用具     |      6800 | NULL    | NULL      |
+------------+--------------+--------------+-----------+---------+-----------+
6 rows in set (0.01 sec)
```

### 4.4分别使用内连结和关联子查询每一类商品中售价最高的商品。

```sql
--内连结
select p1.product_id,p1.product_type,p1.product_name,p1.sale_price,p2.max_price from product as p1 inner join (select product_type,max(sale_price) as max_price from product group by product_type) as p2 on p1.product_type = p2.product_type and p1.sale_price = p2.max_price;
+------------+--------------+--------------+------------+-----------+
| product_id | product_type | product_name | sale_price | max_price |
+------------+--------------+--------------+------------+-----------+
| 0002       | 办公用品     | 打孔器       |        500 |       500 |
| 0003       | 衣服         | 运动T恤      |       4000 |      4000 |
| 0005       | 厨房用具     | 高压锅       |       6800 |      6800 |
+------------+--------------+--------------+------------+-----------+
3 rows in set (0.00 sec)
select p1.product_id,p1.product_name,p1.product_type,p1.sale_price from product as p1 where p1.sale_price = (select max(sale_price) as max_sale_price from product as p2 where p1.product_type = p2.product_type group by product_type);
```

### 4.5用关联子查询实现：在`product`表中，取出 product_id, produc_name, slae_price, 并按照商品的售价从低到高进行排序、对售价进行累计求和。

```sql
select product_id,product_name,sale_price (select sum(sale_price) from product as p2 where p1.sale_price >= p2.sale_price and p1.product_id <= p2.product_id) as '累计销售价格' from product as p1 order by sale_price;
----------------------------
select p1.product_id,p1.product_name,p2.sale_price,sum(p2.sale_price) as '累计销售价格' from product as p1 left join product as p2 on p1.sale_price >= p2.sale_price and p1.product_id != p2.product_id group by p1.product_id,p1.product_type order by sale_price desc;
+------------+--------------+------------+--------------------+
| product_id | product_name | sale_price | 累计销售价格       |
+------------+--------------+------------+--------------------+
| 0003       | 运动T恤      |       1000 |               5980 |
| 0004       | 菜刀         |       1000 |               2980 |
| 0005       | 高压锅       |       1000 |               9980 |
| 0001       | T恤衫        |        500 |               1980 |
| 0006       | 叉子         |        500 |                600 |
| 0007       | 擦菜板       |        500 |               1100 |
| 0002       | 打孔器       |        500 |                600 |
| 0008       | 圆珠笔       |       NULL |               NULL |
+------------+--------------+------------+--------------------+
8 rows in set (0.00 sec)
```













