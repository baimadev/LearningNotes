## AVG()

AVG() 函数返回数值列的平均值。

```sql
SELECT AVG(column_name) FROM table_name
```
下面的 SQL 语句选择访问量高于平均访问量的 "site_id" 和 "count"：

```sql
SELECT site_id, count FROM access_log
WHERE count > (SELECT AVG(count) FROM access_log);
```

## COUNT() 
COUNT() 函数返回匹配指定条件的行数。

以下 SQL 语句的结果是 2，因为客户 Carter 共有 2 个订单：

```sql
SELECT COUNT(Customer) AS CustomerNilsen FROM Orders
WHERE Customer='Carter'
```


以下 SQL 语句返回表的总行数：

```sql
SELECT COUNT(*) AS NumberOfOrders FROM Orders
```

以下 SQL 语句返回"Orders" 表中不同客户的数目。

```sql
SELECT COUNT(DISTINCT Customer) AS NumberOfCustomers FROM Orders
```

## First
FIRST() 函数返回指定的列中第一个记录的值。

```sql
SELECT name AS FirstSite FROM Websites LIMIT 1;
```

## Last

```sql
SELECT name FROM Websites
ORDER BY id DESC
LIMIT 1;
```

## MAX MIN

```sql
SELECT MAX(column_name) FROM table_name;
```
```sql
SELECT MIN(column_name) FROM table_name;
```

## SUM

```sql
SELECT SUM(column_name) FROM table_name;
```

## GROUP BY
GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组。

```sql
SELECT site_id, SUM(access_log.count) AS nums
FROM access_log GROUP BY site_id;
```
先根据site_id分组，然后将每组的count相加。结果返回每个site_id的count总和。

```sql
SELECT Websites.name,COUNT(access_log.aid) AS nums FROM access_log
LEFT JOIN Websites
ON access_log.site_id=Websites.id
GROUP BY Websites.name;
```
先根据Left join匹配，然后根据name分组，然后返回每组的行数。

## Having

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用。

HAVING 子句可以让我们筛选分组后的各组数据。

```sql
SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums FROM (access_log
INNER JOIN Websites
ON access_log.site_id=Websites.id)
GROUP BY Websites.name
HAVING SUM(access_log.count) > 200;
```
先根据inner join匹配，然后根据name分组，然后每组sum累加count，having筛选累加值大于200的。

## EXISTS 

EXISTS 运算符用于判断查询子句是否有记录，如果有一条或多条记录存在返回 True，否则返回 False。

```sql
SELECT Websites.name, Websites.url 
FROM Websites 
WHERE EXISTS (SELECT count FROM access_log WHERE Websites.id = access_log.site_id AND count > 200);
```
WHERE为true则返回结果。

EXISTS 可以与 NOT 一同使用，查找出不符合查询语句的记录：

```sql
SELECT Websites.name, Websites.url 
FROM Websites 
WHERE NOT EXISTS (SELECT count FROM access_log WHERE Websites.id = access_log.site_id AND count > 200);
```

## UCASE()  LCASE(）

把name都转换为大写。

```sql
SELECT UCASE(name) AS site_title, url
FROM Websites;
```

```sql
SELECT LCASE(name) AS site_title, url
FROM Websites;
```

## MID()

下面的 SQL 语句从 "Websites" 表的 "name" 列中提取前 4 个字符：

```sql
SELECT MID(name,1,4) AS ShortTitle
FROM Websites;
```

## LEN() 

下面的 SQL 语句从 "Websites" 表中选取 "name" 和 "url" 列中值的长度：

```sql
SELECT name, LENGTH(url) as LengthOfURL
FROM Websites;
```

## ROUND()

 返回参数X的四舍五入的一个整数。
 
```sql
mysql> select ROUND(-1.23);
        -> -1
mysql> select ROUND(-1.58);
        -> -2
mysql> select ROUND(1.58);
        -> 2
```

ROUND(X,D)：返回参数X的四舍五入的有 D 位小数的一个数字。如果D为0，结果将没有小数点或小数部分。

```sql
mysql> select ROUND(1.298, 1);
        -> 1.3
mysql> select ROUND(1.298, 0);
        -> 1
```

## NOW() 

从 "Websites" 表中选取 name，url，及当天日期：
```sql
SELECT name, url, Now() AS date
FROM Websites;
```

## FORMAT()

下面的 SQL 语句从 "Websites" 表中选取 name, url 以及格式化为 YYYY-MM-DD 的日期：

```sql
SELECT name, url, DATE_FORMAT(Now(),'%Y-%m-%d') AS date
FROM Websites;
```