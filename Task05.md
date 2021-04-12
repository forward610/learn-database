| DATE：2021-04-11     |
| -------------------- |
| **Author**：Fahaxiki |

---

[TOC]

## 5.1窗口函数

窗口函数也称**OLAP**函数。意思是对数据库数据进行实时分析处理

```sql
<窗口函数> OVER ([partition by <列名>] order by <排序用列名>)
```

- **partition by**是用来分组，即选择要看那个窗口，类似于group by子句的分组功能，但是partition by子句并不具备group by自居的汇总函数，并不会改变原始表中记录的行数
- **order by**是用来排序，即决定窗口内，是按那种规则（字段）来排序的

```sql
select product_name,product_type,sale_price,rank() over (partition by product_type order by sale_price) as ranking from product;
+--------------+--------------+------------+---------+
| product_name | product_type | sale_price | ranking |
+--------------+--------------+------------+---------+
| 圆珠笔       | 办公用品     |        100 |       1 |
| 打孔器       | 办公用品     |        500 |       2 |
| 叉子         | 厨房用具     |        500 |       1 |
| 擦菜板       | 厨房用具     |        880 |       2 |
| 菜刀         | 厨房用具     |       3000 |       3 |
| 高压锅       | 厨房用具     |       6800 |       4 |
| T恤衫        | 衣服         |       1000 |       1 |
| 运动T恤      | 衣服         |       4000 |       2 |
+--------------+--------------+------------+---------+
8 rows in set (0.01 sec)
```

## 5.2窗口函数种类

1. 将SUM、MAX、MIN等聚合函数用在窗口函数中
2.  RANK、DENSE_RANK等排序用的专用窗口函数

### 5.2.1专用窗口函数

- **RANK函数**(英式排序)

计算排序时，如果存在相同位次的记录，则会跳过之后的位次。

例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、4 位……

- **DENSE_RANK函数****（中式排序）**

同样是计算排序，即使存在相同位次的记录，也不会跳过之后的位次。

例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、2 位……

- **ROW_NUMBER函数**

赋予唯一的连续位次。

例）有 3 条记录排在第 1 位时：1 位、2 位、3 位、4 位

```sql
select product_name,product_type,sale_price,rank() over (order by sale_price) as ranking,dense_rank() over (order by sale_price) as dense_ranking,row_number() over (order by sale_price) as row_num from product;
+--------------+--------------+------------+---------+---------------+---------+
| product_name | product_type | sale_price | ranking | dense_ranking | row_num |
+--------------+--------------+------------+---------+---------------+---------+
| 圆珠笔       | 办公用品     |        100 |       1 |             1 |       1 |
| 打孔器       | 办公用品     |        500 |       2 |             2 |       2 |
| 叉子         | 厨房用具     |        500 |       2 |             2 |       3 |
| 擦菜板       | 厨房用具     |        880 |       4 |             3 |       4 |
| T恤衫        | 衣服         |       1000 |       5 |             4 |       5 |
| 菜刀         | 厨房用具     |       3000 |       6 |             5 |       6 |
| 运动T恤      | 衣服         |       4000 |       7 |             6 |       7 |
| 高压锅       | 厨房用具     |       6800 |       8 |             7 |       8 |
+--------------+--------------+------------+---------+---------------+---------+
8 rows in set (0.00 sec)
```

### 5.2.2聚合函数在窗口函数上的使用

聚合函数在开窗函数中的使用方法和之前的专用窗口函数一样，只是出来的结果是一个**累计**的聚合函数值

```sql
select product_id,product_name,sale_price,sum(sale_price) over (order by product_id) as current_sum,avg(sale_price) over (order by product_id) as current_avg from product;
```

## 5.3窗口函数的应用-计算移动平均

```sql
--语法
<窗口函数> over (order by <排序用列名> rows n preceding)
<窗口函数> over (order by <排序用列名> rows between n preceding and n following)
/* PRECEDING（“之前”）， 将框架指定为 “截止到之前 n 行”，加上自身行
FOLLOWING（“之后”）， 将框架指定为 “截止到之后 n 行”，加上自身行
BETWEEN 1 PRECEDING AND 1 FOLLOWING，将框架指定为 “之前1行” + “之后1行” + “自身” */
select product_id,product_name,sale_price,avg(sale_price) over (order by product_id rows 2 preceding) as moving_avg,avg(sale_price) over (order by product_id rows between 1 preceding and 1 following) as moving_avg from product;
```

### 5.3.1窗口函数使用范围和注意事项

- 原则上，窗口函数只能在select子句中使用
- 窗口函数over中的order by 子句并不会影响最终结果的排序。其知识用来决定窗口函数按何种顺序计算

## 5.4GROUPING运算符

### 5.4.1    rollup ——计算合计及小计

```sql
--常规的GROUP BY 只能得到每个分类的小计，有时候还需要计算分类的合计，可以用 ROLLUP关键字
select product_type,regist_date,sum(sale_price) as sum_price from product group by product_type,regist_date with rollup;
+--------------+-------------+-----------+
| product_type | regist_date | sum_price |
+--------------+-------------+-----------+
| 办公用品     | 2009-09-11  |       500 |
| 办公用品     | 2009-11-11  |       100 |
| 办公用品     | NULL        |       600 |   <---- 小计（办公用品）
| 厨房用具     | 2008-04-28  |       880 |
| 厨房用具     | 2009-01-15  |      6800 |
| 厨房用具     | 2009-09-20  |      3500 |
| 厨房用具     | NULL        |     11180 |   <---- 小计（厨房用品）
| 衣服         | NULL        |      4000 |
| 衣服         | 2009-09-20  |      1000 |
| 衣服         | NULL        |      5000 |   <---- 小计（衣服）
| NULL         | NULL        |     16780 |   <---- 合计
+--------------+-------------+-----------+
11 rows in set (0.01 sec)
```

## 练习题

### 5.1请说出针对本章中使用的product（商品）表执行如下 SELECT 语句所能得到的结果。

```sql
select product_id,product_name,sale_price,max(sale_price) over (order by product_id) as current_max_price from product;
--按照 product_id 升序排列，计算出截⾄当前⾏的最⾼ sale_price 。
+------------+--------------+------------+-------------------+
| product_id | product_name | sale_price | Current_max_price |
+------------+--------------+------------+-------------------+
| 0001       | T恤衫        |       1000 |              1000 |
| 0002       | 打孔器       |        500 |              1000 |
| 0003       | 运动T恤      |       4000 |              4000 |
| 0004       | 菜刀         |       3000 |              4000 |
| 0005       | 高压锅       |       6800 |              6800 |
| 0006       | 叉子         |        500 |              6800 |
| 0007       | 擦菜板       |        880 |              6800 |
| 0008       | 圆珠笔       |        100 |              6800 |
+------------+--------------+------------+-------------------+
8 rows in set (0.01 sec)
```

### 5.2继续使用product表，计算出按照登记日期（regist_date）升序进行排列的各日期的销售单价（sale_price）的总额。排序是需要将登记日期为NULL 的“运动 T 恤”记录排在第 1 位（也就是将其看作比其他日期都早）

```sql
select product_name,regist_date,sale_price,sum(sale_price) over (order by regist_date,nulls first) as current_sum_price from product;
```

### 5.3思考题

① 窗口函数不指定PARTITION BY的效果是什么？

>**针对排序列进行全局排序**

② 为什么说窗口函数只能在SELECT子句中使用？实际上，在ORDER BY 子句使用系统并不会报错

```sql
SQL的执行顺序 from -->  where --> groupy by --> having --> select --> order by
如果在 where,group by,having使用了窗口函数，就是说明提前进行了一次排序，排序之后再去除纪录、汇总、汇总过滤，第一次排序结果就是错误，没有实际意义。而 order by语句执行顺序在 select 语句之后，可以使用
```

