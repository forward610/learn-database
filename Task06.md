| DATE：2021-04-12     |
| -------------------- |
| **Author**：Fahaxiki |

---

[TOC]

### 习题1

请使用A股上市公司季度营收预测数据集《Income Statement.xls》和《Company Operating.xlsx》和《Market Data.xlsx》，以Market Data为主表，将三张表中的TICKER_SYMBOL为600383和600048的信息合并在一起。只需要显示以下字段。

| **表名**          | **字段名**    |
| :---------------- | :------------ |
| Income Statement  | TICKER_SYMBOL |
| Income Statement  | END_DATE      |
| Income Statement  | T_REVENUE     |
| Income Statement  | T_COGS        |
| Income Statement  | N_INCOME      |
| Market Data       | TICKER_SYMBOL |
| Market Data       | END_DATE_     |
| Market Data       | CLOSE_PRICE   |
| Company Operating | TICKER_SYMBOL |
| Company Operating | INDIC_NAME_EN |
| Company Operating | END_DATE      |
| Company Operating | VALUE         |

```sql
/*将数据表导⼊数据库，根据表结构和字段含义，Company Operating表使⽤sheet-EN(使⽤CN也可以)，
Income Statement表使⽤sheet-General Business，Market Data使⽤sheet-Data。*/
SELECT MarketData.*,
 OperatingData.INDIC_NAME_EN,
 OperatingData.VALUE,
 IncomeStatement.N_INCOME,
 IncomeStatement.T_COGS,
 IncomeStatement.T_REVENUE
 FROM (
 SELECT TICKER_SYMBOL,
 END_DATE,
 CLOSE_PRICE
 FROM `market data`
 WHERE TICKER_SYMBOL IN ('600383','600048') ) MarketData
 LEFT JOIN 
 (SELECT TICKER_SYMBOL,
 INDIC_NAME_EN,
 END_DATE,
 VALUE
 FROM `company operating`
 WHERE TICKER_SYMBOL IN ('600383','600048') ) OperatingData
 ON MarketData.TICKER_SYMBOL = OperatingData.TICKER_SYMBOL
 AND MarketData.END_DATE = OperatingData.END_DATE
 LEFT JOIN 
 (SELECT DISTINCT TICKER_SYMBOL,
 END_DATE,
 T_REVENUE,
 T_COGS,
 N_INCOME
 FROM `income statement`
 WHERE TICKER_SYMBOL IN ('600383','600048') ) IncomeStatement
ON MarketData.TICKER_SYMBOL = IncomeStatement.TICKER_SYMBOL
 AND MarketData.END_DATE = IncomeStatement.END_DATE
ORDER BY MarketData.TICKER_SYMBOL, MarketData.END_DATE
```

### 习题2

请使用 Wine Quality Data 数据集《winequality-red.csv》，找出 pH=3.03的所有红葡萄酒，然后，对其 citric acid 进行中式排名（相同排名的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”）

```sql
-- 窗口函数来排序
select pH,`citric acid`,dense_rank() over (order by `citric acid`) as rankn from `winequality-red` where pH = 3.03;
+------+-------------+-------+
| pH   | citric acid | rankn |
+------+-------------+-------+
| 3.03 | 0.48        |     1 |
| 3.03 | 0.51        |     2 |
| 3.03 | 0.51        |     2 |
| 3.03 | 0.52        |     3 |
| 3.03 | 0.59        |     4 |
| 3.03 | 0.65        |     5 |
+------+-------------+-------+
6 rows in set (0.02 sec)
```

### 习题3

使⽤Coupon Usage Data for O2O中的数据集《ccf_offline_stage1_test_revised.csv》，试分别找出在 2016年7⽉期间，发放优惠券总⾦额最多和发放优惠券张数最多的商家。 这⾥只考虑满减的⾦额，不考虑打⼏折的优惠券。

```sql
		-- 字符串截取数的应用，还有分组聚合函数
-- 发放优惠券总金额最多的商家
select Merchant_id,sum(SUBSTRING_INDEX(`Discount_rate`,':',-1)) as discount_amount from ccf_offline_stage1_test_revised where Date_received Between '2016-07-01' and '2016-07-31' group by Merchant_id order by discount_amount desc limit 1;
+-------------+-----------------+
| Merchant_id | discount_amount |
+-------------+-----------------+
| 760         |            1260 |
+-------------+-----------------+
1 row in set (0.02 sec)
-- 发放优惠券张数最多的商家
select Merchant_id,count(1) as cnt from ccf_offline_stage1_test_revised where Date_received Between '2016-07-01' and '2016-07-31' group by Merchant_id order by cnt desc limit 1;
+-------------+-----+
| Merchant_id | cnt |
+-------------+-----+
| 760         | 252 |
+-------------+-----+
1 row in set (0.00 sec)
```

### 习题4

请使⽤A股上市公司季度营收预测中的数据集《Macro&Industry.xlsx》中的sheet-INDIC_DATA，请计 算全社会⽤电量:第⼀产业:当⽉值在2015年⽤电最⾼峰是发⽣在哪⽉？并且相⽐去年同期增⻓/减少了多 少个百分⽐？

```sql
-- 2015年用电最高峰时发生在哪月
select PERIOD_DATE,max(DATA_VALUE) FianValue from `macro industry` where INDIC_ID = '2020101522' and year(PERIOD_DATE) = 2015 group by PERIOD_DATE order by FianValue desc limit 1;
+-------------+-----------+
| PERIOD_DATE | FianValue |
+-------------+-----------+
| 2015-08-31  | 133.41650 |
+-------------+-----------+
1 row in set (0.05 sec)
-- 并且相⽐去年同期增⻓/减少了多 少个百分⽐？
SELECT BaseData.*,
 (BaseData.FianlValue - YoY.FianlValue) / YoY.FianlValue YoY
 FROM (SELECT PERIOD_DATE,
 MAX(DATA_VALUE) FianlValue
 FROM `macro industry`
 WHERE INDIC_ID = '2020101522'
 AND YEAR(PERIOD_DATE) = 2015
 GROUP BY PERIOD_DATE
 ORDER BY FianlValue DESC
 LIMIT 1) BaseData
 LEFT JOIN -- YOY
 (SELECT PERIOD_DATE,
 MAX(DATA_VALUE) FianlValue
 FROM `macro industry`
 WHERE INDIC_ID = '2020101522'
 AND YEAR(PERIOD_DATE) = 2014
 GROUP BY PERIOD_DATE ) YoY
 ON YEAR(BaseData.PERIOD_DATE) = YEAR(YoY.PERIOD_DATE) + 1
 AND MONTH(BaseData.PERIOD_DATE) = MONTH(YoY.PERIOD_DATE);
 +-------------+------------+-------------+
| PERIOD_DATE | FianlValue | YoY         |
+-------------+------------+-------------+
| 2015-08-31  |  133.41650 | 0.023151500 |
+-------------+------------+-------------+
1 row in set (0.04 sec)
```

### 习题5

使⽤Coupon Usage Data for O2O中的数据集《ccf_online_stage1_train.csv》，试统计在2016年6⽉期间，线上总体优惠券弃⽤率为多少？并找出优惠券弃⽤率最⾼的商家。 弃⽤率 = 被领券但未使⽤的优惠券张数 / 总的被领取优惠券张数

```sql
	-- case when 条件判断语句
-- 2016年6⽉期间，线上总体优惠券弃⽤率为多少？
SELECT SUM(CASE WHEN Date='0000-00-00' AND Coupon_id IS NOT NULL
 THEN 1
 ELSE 0
 END) /
 SUM(CASE WHEN Coupon_id IS NOT NULL
 THEN 1
 ELSE 0
 END) AS discard_rate
FROM ccf_online_stage1_train
WHERE Date_received BETWEEN '2016-06-01' AND '2016-06-30';
-- 2016年6⽉期间，优惠券弃⽤率最⾼的商家？
SELECT Merchant_id,
 SUM(CASE WHEN Date = '0000-00-00' AND Coupon_id IS NOT NULL
 THEN 1
 ELSE 0
 END) /
 SUM(CASE WHEN Coupon_id IS NOT NULL
 THEN 1
 ELSE 0
 END) AS discard_rate
 FROM ccf_online_stage1_train
WHERE Date_received BETWEEN '2016-06-01' AND '2016-06-30'
GROUP BY Merchant_id
ORDER BY discard_rate DESC
LIMIT 1;
+-------------+--------------+
| Merchant_id | discard_rate |
+-------------+--------------+
| 43809       |       0.0000 |
+-------------+--------------+
1 row in set (0.00 sec)
```

### 习题6

请使⽤ Wine Quality Data 数据集《winequality-white.csv》，找出 pH=3.63的所有⽩葡萄酒，然后， 对其 residual sugar 量进⾏英式排名（⾮连续的排名）

```sql
-- 窗口函数
select pH,`regidual sugar`,rank() over (order by `residual sugar`) as rankn from `winequanlity-white` where pH = 3.63;
```

### 习题7

请使⽤A股上市公司季度营收预测中的数据集《Market Data.xlsx》中的sheet-DATA， 计算截⽌到2018年底，市值最⼤的三个⾏业是哪些？以及这三个⾏业⾥市值最⼤的三个公司是哪些？ （每个⾏业找出前三⼤的公司，即⼀共要找出9个）

```sql
		---利用日期函数、窗口函数、子查询、多表查询、分组聚合
--计算截至到2018年底，市值最大的三个行业是那些？
select TYPE_NAME_CN,sum(MARKET_VALUE) from `market data` where year(END_DATE) = '2018-12-31' group by TYPE_NAME_CN order by sum(MARKET_VALUE) desc limit 3;
+--------------+-------------------+
| TYPE_NAME_CN | sum(MARKET_VALUE) |
+--------------+-------------------+
| 非银金融     |      121575467542 |
| 电子         |      114745124524 |
| 交通运输     |      113366400000 |
+--------------+-------------------+
3 rows in set, 1 warning (0.01 sec)
--这三个行业里市值最大的三个公司是那些？
SELECT BaseData.TYPE_NAME_CN,
 BaseData.TICKER_SYMBOL
 FROM (SELECT TYPE_NAME_CN,
 TICKER_SYMBOL,
 MARKET_VALUE,
 ROW_NUMBER() OVER(PARTITION BY TYPE_NAME_CN ORDER BY MARKET_VALUE)
CompanyRanking
 FROM `market data` ) BaseData
 LEFT JOIN
 ( SELECT TYPE_NAME_CN,
 SUM(MARKET_VALUE)
 FROM `market data`
 WHERE YEAR(END_DATE) = '2018-12-31'
 GROUP BY TYPE_NAME_CN
 ORDER BY SUM(MARKET_VALUE) DESC
 LIMIT 3 ) top3Type
 ON BaseData.TYPE_NAME_CN = top3Type.TYPE_NAME_CN
WHERE CompanyRanking <= 3
 AND top3Type.TYPE_NAME_CN IS NOT NULL;
 +--------------+---------------+
| TYPE_NAME_CN | TICKER_SYMBOL |
+--------------+---------------+
| 交通运输     | 603056        |
| 交通运输     | 603056        |
| 交通运输     | 603056        |
| 电子         | 002850        |
| 电子         | 002850        |
| 电子         | 002850        |
| 非银金融     | 601375        |
| 非银金融     | 601375        |
| 非银金融     | 601375        |
+--------------+---------------+
9 rows in set, 1 warning (0.01 sec)
```

### 习题8

使用Coupon Usage Data for O2O中的数据集《ccf_online_stage1_train.csv》和《ccf_offline_stage1_train.csv》，试找出在2016年6月期间，线上线下累计优惠券使用次数最多的顾客

```sql
--利用分组聚合，字符串截取，null值，子查询，行合并
SELECT User_id
,SUM(couponCount) AS couponCount
FROM 
(SELECT User_id
 ,count(*) AS couponCount
FROM `ccf_online_stage1_train`
WHERE (Date IS NOT NULL AND Coupon_id IS NOT NULL)
AND (LEFT(DATE,4)=2016 )
GROUP BY User_id
UNION ALL
SELECT User_id
,COUNT(*) AS couponCount
FROM `ccf_offline_stage1_train`
WHERE (Date IS NOT NULL AND Coupon_id IS NOT NULL)
AND (LEFT(DATE,4)=2016)
GROUP BY User_id) AS a
GROUP BY User_id
ORDER BY SUM(couponCount) DESC
LIMIT 1;
+----------+-------------+
| User_id  | couponCount |
+----------+-------------+
| 12985299 |           2 |
+----------+-------------+
1 row in set (0.01 sec)
```

### 习题9

请使用A股上市公司季度营收预测数据集《Income Statement.xls》中的sheet-General Business和《Company Operating.xlsx》中的sheet-EN。

找出在数据集所有年份中，按季度统计，白云机场旅客吞吐量最高的那一季度对应的**净利润**是多少？（注意，是单季度对应的净利润，非累计净利润。）

```sql
--利用日期函数处理还有子查询连接多表
SELECT *
FROM 
(SELECT TICKER_SYMBOL
,YEAR(END_DATE) AS Year
,QUARTER(END_DATE) AS QUARTER
,SUM(VALUE) AS Amount
FROM `company operating`
WHERE INDIC_NAME_EN = 'Baiyun Airport:Passenger throughput'
GROUP BY TICKER_SYMBOL,YEAR(END_DATE),QUARTER(END_DATE)
ORDER BY SUM(VALUE) DESC
LIMIT 1 ) AS a
LEFT JOIN
(SELECT TICKER_SYMBOL
,YEAR(END_DATE) AS Year
,QUARTER(END_DATE) AS QUARTER
,SUM(N_INCOME) AS Amount
FROM `income statement`
GROUP BY TICKER_SYMBOL,YEAR(END_DATE),QUARTER(END_DATE)) AS i
ON a.TICKER_SYMBOL = i.TICKER_SYMBOL
AND a.Year = i.Year
AND a.QUARTER = i.QUARTER;
+---------------+------+---------+----------+---------------+------+---------+--------+
| TICKER_SYMBOL | Year | QUARTER | Amount   | TICKER_SYMBOL | Year | QUARTER | Amount |
+---------------+------+---------+----------+---------------+------+---------+--------+
| 600004        | 2018 |       1 | 17316571 | NULL          | NULL |    NULL |   NULL |
+---------------+------+---------+----------+---------------+------+---------+--------+
1 row in set (0.02 sec)
```

### 习题10

使用Coupon Usage Data for O2O中的数据集《ccf_online_stage1_train.csv》和《ccf_offline_stage1_train.csv》，试找出在2016年6月期间，线上线下累计被使用优惠券满减最多的前3名商家。

比如商家A，消费者A在其中使用了一张200减50的，消费者B使用了一张30减1的，那么商家A累计被使用优惠券满减51元

```sql
--利用字符串截取还有行合并，分组聚合
SELECT Merchant_id
,SUM(discount_amount) AS discount_amount
FROM
(SELECT Merchant_id
,SUM(SUBSTRING_INDEX(`Discount_rate`,':',-1)) AS discount_amount
FROM `ccf_online_stage1_train`
WHERE (Date IS NOT NULL AND Coupon_id IS NOT NULL)
AND (LEFT(DATE,4)=2016 )
AND MID(DATE,5,2) = '06'
GROUP BY Merchant_id
UNION ALL
SELECT Merchant_id
,SUM(SUBSTRING_INDEX(`Discount_rate`,':',-1)) AS discount_amount
FROM `ccf_offline_stage1_train`
WHERE (Date IS NOT NULL AND Coupon_id IS NOT NULL)
AND (LEFT(DATE,4)=2016 )
AND MID(DATE,5,2) = '06'
GROUP BY Merchant_id) AS a
GROUP BY Merchant_id
ORDER BY SUM(discount_amount) DESC
LIMIT 1;
```

