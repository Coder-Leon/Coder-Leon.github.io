---
layout: post
title: '读书笔记-《MySQL必知必会》'
subtitle: 'MySQL'
date: 2018-05-22
categories: 数据库
tags: 数据库
---

花了半天重新刷完了《MySQL必知必会》这本小书，把SQL复习一下，包括一些要注意的地方。

### 一些需要注意的地方：
- 不能部分使用DISTINCT，DISTINCT应用于所有列而不仅是前置它的列。除非两行的字段数据刚好都一样。例如：
  
  ```SQL
    SELECT DISTINCT vend_id, prod_price FROM products;
  ```
  原始数据：
  
 vend_id | prod_price 
 :-: | :-: 
 1000 | 123 
 1000 | 125 
 2000 | 200 
 2000 | 200 

  查询结果将是：

 vend_id | prod_price 
 :-: | :-: 
 1000 | 123 
 1000 | 125 
 2000 | 200 

- LIMIT index, num 指明返回从第 index 行开始的前 num 行，等同于LIMIT index OFFSET num。第一行是0行，不是1。

- 多个列排序，优先按前一个指定的列排序，除非此列有相同的两行记录。默认是ASC升序排列，可以指定DESC降序排列。如果想对多个列同时降序，则需要在每个字段后指定DESC。MySQL字典排序中，大小写字母视为相同，即A与a没有先后顺序，为同一字典序。

  ```SQL
    ORDER BY prod_price DESC, prod_name DESC
  ```
- <> 等同于 !=

- BETWEEN begin AND end 两值之间[begin, end]闭区间，且必须 end > begin

- 含有NULL值的字段要特别注意

 vend_id | prod_price 
 :-: | :-: 
 1000 | 123  
 1200 | null
 
 >  执行以下SQL时，不会返回含有NULL的行。
 
  ```SQL
    SELECT * FROM products WHERE vend_id > 1000 ;
  ```

- 任何时候使用具有 AND 和 OR 操作符的 WHERE 字句时，都应该用圆括号明确地分组操作符，即使 AND 优先级高于 OR。 

- MySQL中的 NOT 只支持对 IN, BETWEEN 和 EXISTS 取反，这是和其他 DBMS 支持对各种条件取反的差异所在。

- 通配符 % 除了 代表给定位置的一个或多个字符外，还可以代表0个字符。

- 通配符 _ 代表一个字符。

- 尽量减少使用通配符，因为所需时间更长。

- 正则中的 ^ 的两个用途，在集合中 [^ 0-9 ] 用来否定该集合，否则，用来指串的开始处，^ [ 0-9 ] 代表串的开始处为 0-9 。

- 简单的正则测试，总是返回0（不匹配）或1（匹配）。如下返回1。

  ```SQL
    SELECT 'hello' REGEXP '[^0-9]'
  ``` 
- MySQL使用 Concat（）函数来实现拼接，而不是 + 或 \| \| ，例如：

  ```SQL
    SELECT Concat(vend_name, ' (', vend_country, ')') AS vend_title FROM vendors;  
  ```
  
  | vend_title |
  |:-:|
  | HUAWEI  (CHINA) |
 
- MySQL自带去空白函数 Trim（） LTrim（） RTrim（）
> 为了提高 SQL 的可移植性且平时接触的大都是在应用层做处理，感觉还是尽量少在数据库中进行非聚集函数的处理。

- 聚集函数，包括 AVG( )、COUNT( )、MAX( )、MIN( )、SUM( )
> AVG( ) 忽略列值为NULL的行 
> COUNT( * ) 对表中行的数目进行计数，不管列中的值是否为 NULL，即不忽略 NULL
> COUNT( column ) 对特定列具有值的行进行计数，会忽略 NULL
> MAX( ) 、 MIN( ) 与 SUM( )都会忽略对列值 NULL 的计算，虽然好像忽略不忽略不影响结果。

- AVG( ) 可与 DISTINCT 联合使用：\

```SQL
    SELECT AVG(DISTINCT prod_price) AS avg_price FROM products;
```
> 通过去除很多重复的低价商品，来避免拉低均价。

- DISTINCT 与 COUNT( )联合使用时，必须指定列名，不能用于 COUNT( DISTINCT * )

- 使用 GROUP BY 进行数据分组：

```SQL
    SELECT vend_id, COUNT(*)  AS num_prods FROM products GROUP BY vend_id;
```

| vend_id | num_prods |
| :-: | :-: |
| 1001 | 3 |
| 1002 | 2 |
| 1003 | 6 |
| NULL | 2 |

> 使用分组后，是对每个 vend_id 进行 count 计算，而不是整个表，可以看到供应商1001有3个产品，1002有2个商品。。。有两个商品没供应商。
> 如果分组中列中具有 NULL 值，则将 NULL 作为一个分组返回，有多行 NULL 值，则将它们分为一组。
> GROUP BY 必须出现在 WHERE 之后，ORDER BY 之前。
> 在MySQL5.7 中，默认启用了ONLY_FULL_GROUP_BY，和ORDER BY 联合使用的时候要注意。

- 使用 HAVING 过滤分组，因为 WHERE 只过滤行而不是分组。WHERE 是在分组前进行过滤，而 HAVING 是在分组后进行过滤。

```SQL
    SELECT vend_id, COUNT(*)  AS num_prods FROM products GROUP BY vend_id HAVING COUNT(*) >=3;
```

| vend_id | num_prods |
| :-: | :-: |
| 1001 | 3 |
| 1003 | 6 |

- WHERE 与 GROUP BY 联合使用：

```SQL
    SELECT vend_id, COUNT(*)  AS num_prods FROM products WHERE prod_price >=10 GROUP BY vend_id HAVING COUNT(*) >=3;
```

| vend_id | num_prods |
| :-: | :-: |
| 1003 | 6 |

> 因为供应商1001只有两个商品的价格大于10，所以结果里没有列出。

- 嵌套子查询性能较低，应减少使用进而使用性能更好的联结等操作。

- 如果数据存储在多个表中，怎样使用单条 SELECT 语句检索出来，答案是使用联结，用在一条 SELECT 语句中关联多个表，返回一组输出。下面的演示为内部联结，也称等值联结（equijoin）。

```SQL
    SELECT vend_name, prod_name, prod_price FROM vendors, products WHERE vendors.vend_id = products.vend_id;
```

- 笛卡尔积（CROSS JOIN），得到的表的行数即两个表的行数相乘，结果不堪入目。。。了解即可。

- 内部联结（INNER JOIN）即等值联结的另一种语法表示，返回结果和上一个例子一样：

```SQL
    SELECT vend_name, prod_name, prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
```

> 根据 ANSI SQL 规范尽量使用 INNER JOIN ON 代替 多个SELECT FROM IN WHERE （SELECT FROM）的嵌套子查询，且 INNER JOIN的性能通常大于或等于嵌套子查询。

- 自联结。例如发现某商品（其ID为DTNTR）存在问题，想查询出生产该物品的生产商生产的其他物品是否也有问题，子查询如下：

```SQL
    SELECT prod_id, prod_name FROM products WHERE vend_id 
    = (SELECT vend_id FROM products WHERE prod_id = 'DTNTR';
```

> 下面使用自联结优化： 

```SQL
    SELECT p1.prod_id, p1.prod_name FROM products AS p1, products AS p2 
    WHERE p1.vend_id = p2.vend_id AND p2.prod_id = 'DTNTR';
```

- 无论何时对表进行联结，最少有一个列出现在不止一个表中，即被联结的列。

- 外部联结（OUTER JOIN），先看一个内联结的例子：

    > 检索所有下了订单的客户ID和订单号

```SQL
    SELECT customers.cust_id, orders.order_num FROM customers
    INNER JOIN orders ON customers.cust_id = orders.cust_id;
```

|cust_id|order_num|
|:-:|:-:|
|10001|20005|
|10001|20009|
|10003|20006|

> 使用外联结，检索所有客户ID和订单号，包括没下过订单的客户：

```SQL
    SELECT customers.cust_id, orders.order_num FROM  
    customers LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id;
```

|cust_id|order_num|
|:-:|:-:|
|10001|20005|
|10001|20009|
|10002|NULL|
|10003|20006|

> ID为10002的客户没有下过订单，也被查询出来。
> LEFT OUTER JOIN 是指外联结左边的表，即选择左边表的所有行，RIGHT OUTER JOIN同理。

- 下面看一个复杂点的带聚集函数的联结：

```SQL
SELECT customers.cust_name, customers.cust_id, 
COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```
|cust_name|cust_id|num_ord|
|:-:|:-:|:-:|
|Leon|1001|6|
|Flyxinxin|0323|6|
|Mick|2002|0|
|Jack|2003|2|

> 这个例子使用左外部联结来包含所有客户，甚至包括没有下过任何订单的和客户。

- 组合查询 UNION：

> 先看两个单条查询语句：

```SQL
    SELECT vend_id, prod_id, prod_price FROM products
    WHERE prod_price <= 5;
```

|vend_id|prod_id|prod_price|
|:-:|:-:|:-:|
|1003|FC|2.50|
|1002|FU1|3.42|
|1003|TNT1|2.50|


```SQL
    SELECT vend_id, prod_id, prod_price FROM products
    WHERE vend_id IN (1001,1002);
```

|vend_id|prod_id|prod_price|
|:-:|:-:|:-:|
|1001|OL1|8.99|
|1002|FU1|3.42|

```SQL
    SELECT vend_id, prod_id, prod_price FROM products
    WHERE prod_price <= 5
    UNION 
    SELECT vend_id, prod_id, prod_price FROM products
    WHERE vend_id IN (1001,1002);
```

|vend_id|prod_id|prod_price|
|:-:|:-:|:-:|
|1003|FC|2.50|
|1002|FU1|3.42|
|1003|TNT1|2.50|
|1001|OL1|8.99|

> UNION 最少由两条或两条以上 SELECT 语句组成，语句间用 UNION 分隔。
> UNION 每个查询中必须包含相同的列、表达式或聚集函数。
> UNION 会自动去除重复的行，如需要全部行，使用UNION ALL。


《MySQL必知必会》这本书其实只是用来快速复习一下基础的 SQL 语句，接下来还是要好好啃姜承尧大大的《MySQL技术内幕》。


