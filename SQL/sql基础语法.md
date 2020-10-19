
[跳转目录](#1) 
<h3 id="1"></h3>

## CREATE DATABASE  
创建一个名为 "my_db" 的数据库：  
CREATE DATABASE my_db;

## CREATE TABLE 

```sql
CREATE TABLE table_name  
(  
column_name1 data_type(size),  
column_name2 data_type(size),  
column_name3 data_type(size),  
....  
);

```  
column_name 参数规定表中列的名称。

data_type 参数规定列的数据类型（例如 varchar、integer、decimal、date 等等）。

size 参数规定表中列的最大长度。

for instance:  

```sql
CREATE TABLE Persons  
(  
PersonID int,  
LastName varchar(255),  
FirstName varchar(255),  
Address varchar(255),  
City varchar(255)  
);
```  

PersonID 列的数据类型是 int，包含整数。

LastName、FirstName、Address 和 City 列的数据类型是 varchar，包含字符，且这些字段的最大长度为 255 个字符。

## SQL 约束（Constraints）
SQL 约束用于规定表中的数据规则。

如果存在违反约束的数据行为，行为会被约束终止。

约束可以在创建表时规定（通过 CREATE TABLE 语句），或者在表创建之后规定（通过 ALTER TABLE 语句）。  

- NOT NULL - 指示某列不能存储 NULL 值。   
- UNIQUE - 保证某列的每行必须有唯一的值。
- PRIMARY KEY - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
- FOREIGN KEY - 保证一个表中的数据匹配另一个表中的值的参照完整性。
- CHECK - 保证列中的值符合指定的条件。
- DEFAULT - 规定没有给列赋值时的默认值。

### SQL NOT NULL 约束
在默认的情况下，表的列接受 NULL 值.

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

```sql
CREATE TABLE Persons (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255) NOT NULL,
    Age int
);

```
### SQL UNIQUE 约束
UNIQUE 约束唯一标识数据库表中的每条记录。

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 约束拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

当表已被创建时，如需在 "P_Id" 列创建 UNIQUE 约束，请使用下面的 SQL：

```sql
ALTER TABLE Persons
ADD UNIQUE (P_Id)
```

### SQL PRIMARY KEY 约束
PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

### SQL FOREIGN KEY 约束
一个表中的 FOREIGN KEY 指向另一个表中的 UNIQUE KEY(唯一约束的键)。

```sql
CREATE TABLE Orders
(
O_Id int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
P_Id int FOREIGN KEY REFERENCES Persons(P_Id)
)
```
Orders表中的P\_Id指向Persons表中的P\_Id。

### SQL AUTO INCREMENT 
Auto-increment 会在新记录插入表中时生成一个唯一的数字。

我们通常希望在每次插入新记录时，自动地创建主键字段的值。

我们可以在表中创建一个 auto-increment 字段。

```sql
CREATE TABLE Persons
(
ID int NOT NULL AUTO_INCREMENT,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (ID)
)
```


### SQL CHECK 约束
CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会基于行中其他列的值在特定的列中对值进行限制。

下面的 SQL 在 "Persons" 表创建时在 "P_Id" 列上创建 CHECK 约束。CHECK 约束规定 "P_Id" 列必须只包含大于 0 的整数。

```sql
CREATE TABLE Persons
(
P_Id int NOT NULL CHECK (P_Id>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

### SQL DEFAULT 约束
DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

```sql
CREATE TABLE Persons
(
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'Sandnes'
)
```


## SELECT

### 查询表中所有的值。  
SELECT * FROM table\_name;

### 查询列中不重复的值。  
SELECT DISTINCT  column\_name FROM table\_name;

### WHERE 子句用于提取那些满足指定条件的记录。  
SELECT column\_name,column\_name  
FROM table\_name  
WHERE column\_name operator value;  

for instance:   
SELECT * FROM table\_name  WHERE column\_name = "name";  
**数值不要用引号。**  
**请始终使用 IS NULL 来查找 NULL 值。or( IS NOT NULL )**

### SQL AND & OR 运算符
如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。  
如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

SELECT * FROM Websites WHERE country='CN' AND alexa > 50;  
从 "Websites" 表中选取国家为 "CN" 且alexa排名大于 "50" 的所有网站。

SELECT * FROM Websites WHERE country='USA' OR country='CN';  
从 "Websites" 表中选取国家为 "USA" 或者 "CN" 的所有客户。  

SELECT * FROM Websites WHERE alexa > 15 AND (country='CN' OR country='USA');  
 从 "Websites" 表中选取 alexa 排名大于 "15" 且国家为 "CN" 或 "USA" 的所有网站。

### SQL ORDER BY 关键字
ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 DESC 关键字。

SELECT * FROM Websites ORDER BY alexa DESC;  
从 "Websites" 表中选取所有网站，并按照 "alexa" 降序排列。

SELECT * FROM Websites ORDER BY country,alexa;
从 "Websites" 表中选取所有网站，并按照 "country" 和 "alexa" 升序排列。

**DESC降序， ASC升序。**

## INSERT INTO 语句
INSERT INTO 语句用于向表中插入新记录。  
INSERT INTO 语句可以有两种编写形式。

第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：  
INSERT INTO table_name VALUES (value1,value2,value3,...);

第二种形式需要指定列名及被插入的值:  
INSERT INTO Websites (name, url, alexa, country)   
VALUES ('百度','https://www.baidu.com/','4','CN');

我们也可以在指定的列插入数据,会插入一个新行,未传入的值为数据库默认值，**不用指定主键id。**  

## SQL UPDATE 语句
UPDATE 语句用于更新表中已存在的记录。  
**WHERE 子句规定哪条记录或者哪些记录需要更新。如果您省略了 WHERE 子句，所有的记录都将被更新！**  
UPDATE Websites 
SET alexa='5000', country='USA' 
WHERE name='菜鸟教程';

## SQL DELETE 语句
DELETE 语句用于删除表中的行。  
**WHERE 子句规定哪条记录或者哪些记录需要删除。如果您省略了 WHERE 子句，所有的记录都将被删除！**  
DELETE FROM Websites
WHERE name='Facebook' AND country='USA';  
从 "Websites" 表中删除网站名为 "Facebook" 且国家为 USA 的网站。


## MySQL SELECT LIMIT
下面的 SQL 语句从 "Websites" 表中选取头两条记录：  
SELECT * FROM Websites LIMIT 2;

## SQL LIKE 
LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

SELECT * FROM Websites WHERE name LIKE 'G%';  
选取 name 以字母 "G" 开始的所有客户。

SELECT * FROM Websites WHERE name LIKE '%k';  
选取 name 以字母 "k" 结尾的所有客户。 

SELECT * FROM Websites WHERE name NOT LIKE '%oo%';  
选取 name 不包含模式 "oo" 的所有客户。

## SQL通配符
  
### 通配符描述
%	替代0个或多个字符  
_	替代一个字符  
[charlist]	字符列中的任何单一字符  
[^charlist]或[!charlist] 不在字符列中的任何单一字符  

取 name 以一个任意字符开始，然后是 "oogle" 的所有客户：  
SELECT * FROM Websites WHERE name LIKE '_oogle';  

**MySQL 中使用 REGEXP 或 NOT REGEXP 运算符 (或 RLIKE 和 NOT RLIKE) 来操作正则表达式。**  

选取 name 以 "G"、"F" 或 "s" 开始的所有网站：  
SELECT * FROM Websites WHERE name REGEXP '^[GFs]';

选取 name 以 A 到 H 字母开头的网站：  
SELECT * FROM Websites WHERE name REGEXP '^[A-H]';  

选取 name 不以 A 到 H 字母开头的网站：  
SELECT * FROM Websites WHERE name REGEXP '^[^A-H]';

## SQL IN 操作符
IN 操作符允许您在 WHERE 子句中规定多个值。  
选取 name 为 "Google" 或 "菜鸟教程" 的所有网站：  
SELECT * FROM Websites WHERE name IN ('Google','菜鸟教程');  

## SQL BETWEEN 操作符
BETWEEN 操作符选取介于两个值之间的数据范围内的值。这些值可以是数值、文本或者日期。  

选取 alexa 介于 1 和 20 之间的所有网站：  
SELECT * FROM Websites WHERE alexa BETWEEN 1 AND 20;

如需显示不在上面实例范围内的网站，请使用 NOT BETWEEN：  
SELECT * FROM Websites WHERE alexa NOT BETWEEN 1 AND 20; 

选取 alexa 介于 1 和 20 之间但 country 不为 USA 和 IND 的所有网站：  
SELECT * FROM Websites  
WHERE (alexa BETWEEN 1 AND 20)  
AND country NOT IN ('USA', 'IND'); 

## SQL 别名
通过使用 SQL，可以为表名称或列名称指定别名。  
基本上，创建别名是为了让列名称的可读性更强。

下面的 SQL 语句指定了两个别名，一个是 name 列的别名，一个是 country 列的别名。  
SELECT name AS n, country AS c FROM Websites;    

查询两个表，结果为select的column，使用别名更加简洁。   
SELECT w.name, w.url, a.count, a.date  
FROM Websites AS w, access_log AS a  
WHERE a.site_id=w.id and w.name="菜鸟教程";  

## SQL INNER JOIN 关键字
INNER JOIN 关键字在表中存在至少一个匹配时返回行。

返回满足条件Websites.id=access_log.site_id的所有行，如果不满足则返回空的。   
SELECT Websites.name, access_log.count, access_log.date  
FROM Websites  
INNER JOIN access_log  
ON Websites.id=access_log.site_id  
ORDER BY access_log.count;  

## SQL LEFT JOIN 关键字
LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。  

左表所有的都会列出来，右表没匹配成功则返回null。  
SELECT Websites.name, access_log.count, access_log.date  
FROM Websites  
LEFT JOIN access_log  
ON Websites.id=access_log.site_id  
ORDER BY access_log.count DESC;  

## SQL RIGHT JOIN 关键字
与LEFT JOIN相反。  

## SQL FULL OUTER JOIN 关键字
FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.  
FULL OUTER JOIN 关键字结合了 LEFT JOIN 和 RIGHT JOIN 的结果。

## SQL UNION 操作符
SQL UNION 操作符合并两个或多个 SELECT 语句的结果。  
请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。  
**默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。**  

从 "Websites" 和 "apps" 表中选取所有不同的country（只有不同的值）：  
SELECT country FROM Websites  
UNION  
SELECT country FROM apps  
ORDER BY country;  

## SQL SELECT INTO 语句
通过 SQL，您可以从一个表复制信息到另一个表。  
SELECT INTO 语句从一个表复制数据，然后把数据插入到另一个**新表**中。  

创建 Websites 的备份复件：  
SELECT *  
INTO WebsitesBackup2016  
FROM Websites;  

只复制一些列插入到新表中：  
SELECT name, url  
INTO WebsitesBackup2016  
FROM Websites;  

只复制中国的网站插入到新表中：  
SELECT *  
INTO WebsitesBackup2016  
FROM Websites  
WHERE country='CN';  

复制多个表中的数据插入到新表中：  
SELECT Websites.name, access_log.count, access_log.date  
INTO WebsitesBackup2016  
FROM Websites  
LEFT JOIN access_log  
ON Websites.id=access_log.site_id;  

提示：SELECT INTO 语句可用于通过另一种模式创建一个新的空表。只需要添加促使查询没有数据返回的 WHERE 子句即可：  
SELECT *  
INTO newtable  
FROM table1  
WHERE 1=0;  

## SQL INSERT INTO SELECT 语句

INSERT INTO SELECT 语句从一个表复制数据，然后把数据插入到一个**已存在的表**中。     
 
复制 "apps" 中的数据插入到 "Websites" 中：  
INSERT INTO Websites (name, country)  
SELECT app_name, country FROM apps;


## SQL CREATE INDEX 语句
CREATE INDEX 语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

在表上创建一个简单的索引。允许使用重复的值：

```sql
CREATE INDEX index_name
ON table_name (column_name)
```

在表上创建一个唯一的索引。不允许使用重复的值：唯一的索引意味着两个行不能拥有相同的索引值。

```sql
CREATE UNIQUE INDEX index_name
ON table_name (column_name)
```

**注释：用于创建索引的语法在不同的数据库中不一样。因此，检查您的数据库中创建索引的语法。**

下面的 SQL 语句在 "Persons" 表的 "LastName" 列上创建一个名为 "PIndex" 的索引：

```sql
CREATE INDEX PIndex
ON Persons (LastName)
```
如果您希望索引不止一个列，您可以在括号中列出这些列的名称，用逗号隔开：

```sql
CREATE INDEX PIndex
ON Persons (LastName, FirstName)
```

##DROP

DROP TABLE 语句用于删除表。  
DROP TABLE table_name  

DROP DATABASE 语句用于删除数据库。  
DROP DATABASE database_name  

如果我们仅仅需要删除表内的数据，但并不删除表本身，那么我们该如何做呢？  
TRUNCATE TABLE table_name  

## SQL ALTER TABLE 语句
ALTER TABLE 语句用于在已有的表中添加、删除或修改列。
如需在表中添加列，请使用下面的语法:

```sql
ALTER TABLE table_name
ADD column_name datatype
```

如需删除表中的列，请使用下面的语法（请注意，某些数据库系统不允许这种在数据库表中删除列的方式）：

```sql
ALTER TABLE table_name
DROP COLUMN column_name
```
