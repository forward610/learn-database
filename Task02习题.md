| DATE：2021-04-02     |
| -------------------- |
| **Author**：Fahaxiki |

---

[TOC]

## 第一部分

1. 编写一条SQL语句，从product（商品）表中选取出“登记日期（regist在2009年4月28日之后”的商品，查询结果要包含product_name和regist_date两列。

```sql
select product_name,regist_date from product where regist_date > '2009-04-28';
```

  2.请说出对product 表执行如下3条SELECT语句时的返回结果。

```sql
select * from product where purchase_price = NULL;
答：查询product表中进价为null的
```

```sql
select * from product where purchase_price <> NULL;
答：查询product表中进价不等于null的
```

```sql
select * from prodcut where product_name > NULL;
答：查询product表中商品名称大于null的
```

  3.代码清单2-22（2-2节）中的select语句能够从product表中取出“销售单价（saleprice）比进货单价（purchase price）高出500日元以上”的商品。请写出两条可以得到相同结果的select语句。执行结果如下所示。

| product_name | sale_price | purchase_price |
| ------------ | ---------- | -------------- |
| T恤衫        | 1000       | 500            |
| 运动T恤      | 4000       | 2800           |
| 高压锅       | 6800       | 5000           |

```sql
--第一种
select product_name,sale_price,purchase_price from product where sale_price - purchase_price >= 500;
--第二种
select product_name,sale_price,purchase_price from product where not sale_price-purchase_price < 500;
```

  4.请写出一条select语句，从product表中选取出满足“销售单价打九折之后利润高于100日元的办公用品和厨房用具”条件的记录。查询结果要包括product_name列、product_type列以及销售单价打九折之后的利润（别名设定为profit）。

提示：销售单价打九折，可以通过saleprice列的值乘以0.9获得，利润可以通过该值减去purchase_price列的值获得。

```sql
select product_name,product_type,sale_price as profit from product where sale_price * 0.9 - purchase_price and (product_type = '办公用品' or product_type = '厨房用具');
```

## 第二部分

   5.请指出下述SELECT语句中所有的语法错误。

```sql
select product_id,sum(product_name) from product group by product_type where regist_date > '2009-09-01';
答： 字符型字段product_name不可以进行sum聚合、where应该在 group by的前面 from之后、 group by的聚合键(product_type) 与 select 的子句不同（prodcut_id）
```

  6.![image-20210403221103109](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210403221103109.png)

```sql
select product_type,sum(sale_price),sum(purchase_price) from product group by product_type having sum(sale_price) > sum(purchase_price) * 1.5;
```

  7.![image-20210403224727990](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210403224727990.png)

```sql
select * from product order by regist_date desc,sale_price;
```

