| Date: 2021-04-02     |
| -------------------- |
| **Author**: Fahaxiki |

---

[TOC]

## 一、select语句

### 1.1书写规则

>星号* 代表全部列的意思
>
>SQL中可以随意使用换行符，不影响语句执行（不可插入空行）
>
>设定汉语别名时需要使用双引号括起来
>
>在select语句中使用**distinct**可以删除重复行
>
>注释是SQL语句中标识说明或注意事项的部分。分为1行注释'--'和多行注释两种"/**/" 

### 1.2从表中查询数据

```sql
--语法
select <列名> from <表名>;
--查询全部列时，可以用代表所有列的星号 *
select * from <表名>;
--SQL语句中可以使用as关键字为列设定别名（若中尉需要双引号）
select product_id as id,product_name ad name,purchase_price as '进货单价' from product;
--使用distinct删除product_type列中重复的数据
select distinct product_type from product;
```

### 1.3 where语句

当**查询**‘商品种类为衣服’、‘销售单价在1000以上’等**某些条件的数据时**，使**用where语句**

```sql
select <列名>，... from <表名> where <条件表达式>;
```

## 二、算数、比较运算符

### 2.1 算数运算符

- SQL语句中可以使用四则运算符             **+   -     *      /**

### 2.2 比较运算符

- SQL常见比较运算符

| 运算符 |   含义   |
| :----: | :------: |
|   =    |  和相等  |
|   <>   | 和不相等 |
|   >=   | 大于等于 |
|   >    |   大于   |
|   <=   | 小于等于 |
|   <    |   小于   |

```sql
--查询出sale_price列为500的记录
select product_name,product_type from product sale_price = 500;
```

### 2.3常用法则

1. select字句中可以使用常数或者表达式
2. 使用比较运算符时一定要**注意不等号**和**等号的位置**
3. 字符串类型的数据原则上按照字典顺序进行排序，不能与数字的大小顺序混淆
4. 希望**选取NULL**记录时，需要在条件表达式中使用**IS NULL**运算符。反之**不是NULL**记录，需使用**IS NOT NULL**运算符

```sql
-- SQL语句中使用运算表达式
select product_name,sale_price,sale_price * 2 as 'sale_price ×2' form product;
-- where子句的条件表达式中也可以使用计算表达式
select product_name,sale_price,purchase_price from product where sale_price-purchase_price >= 500;
/*对字符串使用不等号先创建chars并插入数据选取出大于'2'的select语句*/
create table chars (chr char(3) not null,primary key (chr));
-- 查询出大于'2'的数据的select语句
select chr from chars where chr > '2';
-- 查询null的记录
select product_name,purchase_price from product where purchase_price is null;
--查询不为null的记录
select product_name,purchase_price from product where purchase_price is not null;
```

## 三、逻辑运算符

### 3.1 not运算符

- 表示不是...时，可以用not 或 <>,但not范围更广，且**not不能单独使用**

    ```sql
    --选取出销售单价大于等于1000的记录
    select product_name,product_type,sale_price from product where sale_price >= 1000;
    -- 添加not运算符
    select product_name,product_type,sale_price from product where not sale_price >= 1000;
    ```


### 3.2 and 和 or 运算符

- and 相当于'并且'    or 相当于'或者'

>**真值表**
>
>AND：两侧的真值为真返回真，反之假
>
>OR：两侧一个为真返回真，两侧都是假返回假
>
>NOT：单纯的将真换为假，将假换为真

## 四、对表进行聚合查询

### 4.1 聚合函数

- **SQL中用于汇总的函数**叫做**聚合函数**
    - **count**：计算表中多少记录（行数）
    - **sum**：计算表中数值列中数据的总和
    - **avg**：计算表中数值列中的平均值
    - **max**：求出表中任意列中数值的最大值
    - **min**：求出表中任意列中数据的最小值

### 4.2 使用聚合函数删除重复值

```sql
--计算去重数据后的数据行数
select count(distinct product_type) from product;
--是否使用distinct时的动作差异（sum函数）
select sum(sale_price),sum(distinct sale_price) from product;
```

### 4.3 常用法则

1. count函数的结果根据参数的不同而不同。count(*)会得到包含NULL的数据行数，而count(列名)会得到NULL之外的数据行数
2. 聚合函数会将NULL排除在外。但count(*)例外，并不会排除NULL
3. max/min函数几乎适用于所有数据类型的列。sum/avg函数只适用于数值类型的列
4. 想要计算值的种类时，可以在count函数的参数中使用distinct
5. 在聚合函数的参数中使用distinct，可以删除重复数据

## 五 分组查询

### 5.1 group by 语句

```sql
--语法
select <列名1>，<>,<>... from <表名> group by <列名>，<>,<>...;
--示例
--按照商品类统计数据行数
select product_type,count(*) from product group by product_type;
--不用group by区别
select product_type,count(*) from product;
/* GROUP BY 子句就像切蛋糕那样将表进行了分组。在 GROUP BY 子句中指定的列称为聚合键或者分组列。
```

### 5.2聚合键中包含NULL时

```sql
--将进货单价作为聚合键
select purchase_price,count(*) from product group by purchase_price;
```

![](C:\Users\李向前\AppData\Roaming\Typora\typora-user-images\image-20210402231817336.png)

- 此时将NULL作为一组特殊数据进行处理

### 5.3 group by 书写规则

>select ——> from ——> where ——> group by
>
>前三项用于筛选数据，group by对筛选出的数据进行处理

```sql
--where子句中使用group by
select purchase_price, count(*) from product where product_type = '衣服' group by purchase_price;
```

### 5.4 常见错误

- select 子句中如果出现列名，只能是group by子句指定的列名
- 在group by子句中使用列的名字  select子句中可以通过ad来指定别名，但在group by中不能使用别名。因为在DBMS中，select子句在group by子句后执行
- 在where中使用去和函数 原因时聚合函数的使用前提时结果集已经确定，而where还处于确定结果集的过程中，所以相互矛盾会引发错误。如果向指定条件，可以在select，having以及order by子句中使用聚合函数

## 六、聚合结果指定条件

### 6.1 用having得到特定分组

- having子句用于对分组进行过滤，可以使用数字、聚合函数和group by中指定的列名（聚合键）

```sql
--数字
select product_type,count(*) from product group by product_type having count(*) = 2;
-- 错误点（因为product_name不包含在group by 聚合键中）
select product_type,count(*) from product group by product_type having product_name = '圆珠笔';
```

## 七、对查询结果进行排序

### 7.1 order by

```sql
--语法
select <列名1>,<>,<>... from <表名> order by <排序基准列1>,<>,<>...;
```

- **默认为升序排列，降序排列用DESC**

```sql
--降序排列
select product_id,product_name,sale_price,purchase_price from product order by sale_price desc;
--多个排序键
select product_id,product_name,sale_price,purchase_price from product order by sale_price,product_id;
--当用于排序的列名中含有NULL时，NULL会在开头或末尾进行汇总
selct product_id,product_name,sale_price,purchase_price from product order by purchase_price;
```

### 7.2 order by 中列名可使用别名

GROUP BY 子句中不能使用SELECT 子句中定义的别名，但是在 ORDER BY 子句中却可以使用别名。为什么在GROUP BY中不可以而在ORDER BY中可以呢？**执行顺序如下**

>**from** ——> **where** ——> **group** **by** ——> **having** ——> **select** ——> **order by**

