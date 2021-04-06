| DATE：2021-04-04     |
| -------------------- |
| **Auhtor**：Fahaxiki |

---

## 第一部分

1. 创建出满足下述三个条件的视图（视图名称为 ViewPractice5_1）。使用 product（商品）表作为参照表，假设表中包含初始状态的 8 行数据。

    - 条件 1：销售单价大于等于 1000 日元。
    - 条件 2：登记日期是 2009 年 9 月 20 日。
    - 条件 3：包含商品名称、销售单价和登记日期三列。

    对该视图执行 SELECT 语句的结果如下所示。

    ```sql
    select * from ViewPractice5_1;
    --执行结果
    product_name   |  sale_price  | regist_date
    T恤衫				1000		 2009-09-20
    菜刀				3000		 2009-09-20
    ```

    ```sql
    create view ViewPractice5_1 (product_name,sale_price,regist_date) as select product_name,sale_price,regist_date from product where sale_price >= 1000 and regist_date = '2009-09-20'; 
    ```

2. 向习题一中创建的视图 ViewPractice5_1 中插入如下数据，会得到什么样的结果呢？

    ```sql
    insert into ViewPractice5_1 values ('刀子',300,'2009-11-02');
    答：插⼊时将会报错。
    视图插⼊数据时，原表也会插⼊数据，⽽原表数据插⼊时不满⾜约束条件，所以会报错。（因为
    ViewPractice5_1 的原表有三个带有 not null 约束的字段）
    ```

    

3. 请根据如下结果编写 SELECT 语句，其中 sale_price_all 列为全部商品的平均销售单价。

    ```sql
    product_id | product_name | product_type | sale_price | sale_price_all
    ------------+-------------+--------------+------------+---------------------
    0001       | T恤衫         | 衣服         | 1000       | 2097.5000000000000000
    0002       | 打孔器        | 办公用品      | 500        | 2097.5000000000000000
    0003       | 运动T恤       | 衣服          | 4000      | 2097.5000000000000000
    0004       | 菜刀          | 厨房用具      | 3000       | 2097.5000000000000000
    0005       | 高压锅        | 厨房用具      | 6800       | 2097.5000000000000000
    0006       | 叉子          | 厨房用具      | 500        | 2097.5000000000000000
    0007       | 擦菜板        | 厨房用具       | 880       | 2097.5000000000000000
    0008       | 圆珠笔        | 办公用品       | 100       | 2097.5000000000000000
    ```

    ```sql
    select product_id,product_name,product_type,sale_price,(select avg(sale_price) from product) as sale_price_all from product;
    ```

4. 请根据习题一中的条件编写一条 SQL 语句，创建一幅包含如下数据的视图（名称为AvgPriceByType）。

    提示：其中的关键是 avg_sale_price 列。与习题三不同，这里需要计算出的 是各商品种类的平均销售单价。这与使用关联子查询所得到的结果相同。 也就是说，该列可以使用关联子查询进行创建。问题就是应该在什么地方使用这个关联子查询。

    ```sql
    product_id | product_name | product_type | sale_price | avg_sale_price
    ------------+-------------+--------------+------------+---------------------
    0001       | T恤衫         | 衣服         | 1000       |2500.0000000000000000
    0002       | 打孔器         | 办公用品     | 500        | 300.0000000000000000
    0003       | 运动T恤        | 衣服        | 4000        |2500.0000000000000000
    0004       | 菜刀          | 厨房用具      | 3000        |2795.0000000000000000
    0005       | 高压锅         | 厨房用具     | 6800        |2795.0000000000000000
    0006       | 叉子          | 厨房用具      | 500         |2795.0000000000000000
    0007       | 擦菜板         | 厨房用具     | 880         |2795.0000000000000000
    0008       | 圆珠笔         | 办公用品     | 100         | 300.0000000000000000
    ```

    ```sql
    create view AvgPriceByType as select product_id,product_name,product_type,sale_price,(select avg(asle_price) from product as p2 where p1.product_type = p2.product_type group by p1.product_type) as avg_sale_price from product p1;
    ```

## 第二部分

1. 运算或者函数中含有 NULL 时，结果全都会变为NULL ？（判断题）

    - 正确

2. 对本章中使用的 product（商品）表执行如下 2 条 SELECT 语句，能够得到什么样的结果呢？

    ①

    ```sql
    SELECT product_name, purchase_price
      FROM product
     WHERE purchase_price NOT IN (500, 2800, 5000);
     --结果如下
     +--------------+----------------+
    | product_name | purchase_price |
    +--------------+----------------+
    | 打孔器       |            320 |
    | 擦菜板       |            790 |
    +--------------+----------------+
    2 rows in set (0.00 sec)
    --
    解析：该查询语句仅仅取出了 purchase_price 不是 500、2800、5000 的商品，⽽不包含
    purchase_price 为 NULL 的商品，这是因为 谓词⽆法与 NULL 进⾏⽐较。
    ```

    ②

    ```sql
    SELECT product_name, purchase_price
      FROM product
     WHERE purchase_price NOT IN (500, 2800, 5000, NULL);
    --
    mysql> SELECT product_name, purchase_price
        ->   FROM product
        ->  WHERE purchase_price NOT IN (500, 2800, 5000, NULL);
    Empty set (0.00 sec)
    ---
    解析：代码执⾏之前，你可能会认为该语句会返回和查询 ① 同样的结果，实际上它却返回了零条记录，
    这是因为 NOT IN 的参数中不能包含 NULL ，否则，查询结果通常为空。
    ```

3. 按照销售单价（ sale_price）练习 product（商品）表中的商品进行如下分类。

    - 低档商品：销售单价在1000日元以下（T恤衫、办公用品、叉子、擦菜板、 圆珠笔）
    - 中档商品：销售单价在1001日元以上3000日元以下（菜刀）
    - 高档商品：销售单价在3001日元以上（运动T恤、高压锅）

    请编写出统计上述商品种类中所包含的商品数量的 SELECT 语句，结果如下所示。

    执行结果

    ```sql
    low_price | mid_price | high_price
    ----------+-----------+------------
            5 |         1 |         2
    ```

```sql
select 
       sum(case when sale_price <= 1000 then 1 else 0 end) as low_price,
       sum(case when sale_price >= 3001 then 1 else 0 end) as high_price,
       sum(case when sale_price between 1001 and 3000 then 1 else 0 end) as min_price 
from product;
```



