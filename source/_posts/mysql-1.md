---
title: MySQL - 简单介绍 & 类型
date: 2019-08-19 18:28:42
categories: MySQL
tags: 
	- MySQL
---
### SQL简介
SQL 是Structure Query Language（结构化查询语言）的缩写，它是使用关系模型的数据库应用语言。SQL的扩展语言有MySQL、SQL Service等。  
<!-- more -->
SQL 语句主要可以划分为以下3 个类别：
- DDL（Data Definition Languages）语句：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括create、drop、alter等。  
-  DML（Data Manipulation Language）语句：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括insert、delete、udpate 和select 等。
-  DCL（Data Control Language）语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括grant、revoke 等。

### MySQL在线帮助文档
- HELP contents 查看MySQL命令的使用。
其中显示所有可供查询的的分类。  
当需要具体到某个类的时候，可以使用HELP 类别名。eg：`HELP 'Data Type'` 查看所有的数据类型的使用方法。根据全部的一级一级的筛选查询自己想要的内容
- 可以通过SHOW VARIABLES语句查看系统变量及其值。eg:`SHOW VARIABLES like '%show%'`
- 在MySQL中，我们可以使用SHOW STATUS指令语句来查看MySQL服务器的状态信息。当我们希望能够「按需查看」一部分状态信息。这个时候，我们可以在show status语句后加上对应的like子句。例如，我们想要查看当前MySQL启动后的运行时间，我们可以执行如下语句：`show status like '%select%'`
- 查看MySQL版本号：`select version()`

### MySQL 支持的数据类型
#### 整型介绍
###### 整型宽度
对于整型数据，MySQL还支持在类型名称后面的小括号内指定显示宽度，例如int(5)表示当数值宽度小于5位的时候在数字前面填满宽度，如果不显示指定宽度则默认为int(11)。设置了宽度限制后，如果插入大于宽度限制的值，会不会截断或者插不进去报错？答案是肯定的：不会对插入的数据有任何影响，还是按照类型的实际精度进行保存. 
###### UNSIGNED属性
所有的整数类型都有一个可选属性UNSIGNED（无符号），如果需要在字段里面保存非负数或者需要较大的上限值时，可以用此选项，它的取值范围是正常值的下限取0，上限取原值的2倍，例如，tinyint有符号范围是-128～+127，而无符号范围是0～255。。如果一个列指定为zerofill，则MySQL自动为该列添加UNSIGNED属性.
###### AUTO_INCREMENT属性
整数类型还有一个属性：AUTO_INCREMENT。在需要产生唯一标识符或顺序值时，
可利用此属性，这个属性只用于整数类型。AUTO_INCREMENT 值一般从1 开始，每行增加1。
在插入NULL 到一个AUTO_INCREMENT 列时，MySQL 插入一个比该列中当前最大值大1 的
值。一个表中最多只能有一个AUTO_INCREMENT列。对于任何想要使用AUTO_INCREMENT 的
列，应该定义为NOT NULL，并定义为PRIMARY KEY 或定义为UNIQUE 键
###### sql_mode使用
有些时候在你添加数据的时候，明明会出现数据溢出的问题，却依然能正常的操作，只是数据却不是自己想要的。这个时候可以看一下sql_mode的设置。sql_mode的基本命令命令`select @@sql_mode`  `set @@sql_mode=TRADITIONAL`。[使用方法](https://www.cnblogs.com/fireporsche/p/8618691.html)
#### 小数介绍  
###### 分类 
MySQL 分为两种方式：浮点数和定点数。浮点数包括float（单精度）和double（双精度），而定点数则只有decimal一种表示。  
定点数在MySQL内部以字符串形式存放，比浮点数更精确，适合用来表示货币等精度高的数据。  
###### 使用
浮点数和定点数都可以用类型名称后加“(M,D)”的方式来进行表示，“(M,D)”表示该值一共显示M 位数字（整数位+小数位），其中D位位于小数点后面，M 和D 又称为精度和标度。  
MySQL 保存值时进行四舍五入，因此如果在float(7,4)列内插入999.00009，近似结果是999.0001。值得注意的是，浮点数后面跟“(M,D)”的用法是非标准用法，如果要用于数据库的迁移，则最好不要这么使用。
###### 注意
float 和double在不指定精度时，默认会按照实际的精度（由实际的硬件和操作系统决定）
来显示，而decimal在不指定精度时，默认的整数位为10，默认的小数位为0。
#### BIT（位）类型
###### 介绍
对于BIT（位）类型，用于存放位字段值，BIT(M)可以用来存放多位二进制数，M 范围从1～64，如果不写则默认为1位。  
###### 使用
对于位字段，如果直接使用SELECT命令看不到结果，可以用bin()（显示为二进制格式）或者hex()（显示为十六进制格式）函数进行读取
#### 时间类型
###### 分类

类型 | 字节 |最小值 | 最大值|零值表示
---|---|---|---|--
DATE | 4 | 1000-01-01|9999-12-31|0000-00-00
DATETIME | 8|1000-01-01 00:00:00|9999-12-31 23:59:59|0000-00-00 00:00:00
TIMESTAMP|4|19700101080001|2038年的某个时刻|00000000000000 
TIME|3|-838:59:59|838:59:59|00:00:00
YEAR|1|1901|2155|0000

这些数据类型的主要区别如下：
- DATE来表示年月。
- DATETIME表示年月日时分秒。
- TIME 来表示时分秒。
- 通常使用TIMESTAMP经常插入或者更新日期为当前系统时间。TIMESTAMP 值返回后显示为“YYYY-MM-DD HH:MM:SS”格式的字符串，显示宽度固定为19个字符。如果想要获得数字值，应在TIMESTAMP列添加+0。
- YEAR表示年份，它比DATE占用更少的空间。YEAR有2位或4位格式的年。默认是4位格式。在4位格式中，允许的值是1901～2155 和0000。在2位格式中，允许的值是70～69，表示从1970～2069年。MySQL以YYYY 格式显示YEAR值。
###### 使用
DATETIME和TIMESTAMP可以设置CURRENT_TIME默认值为当前系统时间。在添加当前时间时，使用函数now()。

```SQL
insert into t1(id7) values(now());
alter table t1 modify `id5` timestamp NULL DEFAULT CURRENT_TIMESTAMP;
```
###### TIMESTAMP和DATETIEM区别
- TIMESTAMP的插入和查询都受当地时区的影响，更能反应出实际的日期。而DATETIME则只能反应出插入时当地的时区，其他时区的人查看数据必然会有误差的。  
- 案例解释：`show variables like 'time_zone'`当时的时区为SYSTEM，即东八区。时区改为东九区，添加数据，两个字段的属性如下**'id5' timestamp NULL DEFAULT CURRENT_TIMESTAMP,
'id6' datetime DEFAULT CURRENT_TIMESTAMP**
此时两个时间是一致的，都是东九区的时间；然后将时区改为原先的东八区，发现id5时间又变成了东八区的时间，而id6还是原先的九区时间。
###### 扩展
- 查看时区命令

```SQL
show variables like 'time_zone';
select @@time_zone;
```
时区的值为“SYSTEM”，这个值默认是和主机的时区值一致的，因为我们在中国，这里的“SYSTEM”实际是东八区（+8:00）。  
- 修改时区的命令

```SQL
set time_zone='+9:00';
```
### 字符串类型
###### CHAR & VARCHAR
CHAR 和VARCHAR 很类似，都用来保存MySQL 中较短的字符串。两者的存储数据范围不一样。另外在存数据的时候，CHAR列删除了尾部的空格，而VARCHAR 则保留这些空格。
eg:

```SQL
CREATE TABLE vc (v VARCHAR(4), c CHAR(4));
INSERT INTO vc VALUES ('ab ', 'ab ');
```
在添加的时候v是ab&nbsp;&nbsp;而c是ab。   

查看长度：
```SQL
select length(v),length(c) from vc;
```
length(v) | length(c)
---|---
4 | 2

### 函数
###### 字符串函数

函数 | 功能 | 注意点
---|---|---
CANCAT(S1,S2,…Sn) | 连接S1,S2,…Sn 为一个字符串 | 连接的内容有个字段为空，则整个为空
concat_ws(separator, str1, str2, ...)|和concat()一样，将多个字符串连接成一个字符串，但是可以一次性指定分隔符|select concat_ws('-',car_manage_no,car_manage_id) from t1 把分隔符指定为null，结果全部变成了null
group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )|将group by产生的同一个分组中的值连接起来，返回一个字符串结果。|select warehouse_no, warehouse_name, group_concat(concat(plate_city_name,'-',valid_time) order by valid_time desc) from fcm_warehouse_city_valid group by warehouse_no
INSERT(str,x,y,instr) | 将字符串str 从第x 位置开始，y 个字符长的子串替换为字符串instr
LOWER(str)|将字符串str 中所有字符变为小写
UPPER(str)|将字符串str 中所有字符变为大写
LEFT(str ,x)|返回字符串str 最左边的x 个字符|如果第二个参数是NULL，那么将不返回任何字符串
RIGHT(str,x)|返回字符串str 最右边的x 个字符|如果第二个参数是NULL，那么将不返回任何字符串
LPAD(str,n ,pad)|用字符串pad 对str 最左边进行填充，直到长度为n 个字符长度
RPAD(str,n,pad)|用字符串pad 对str 最右边进行填充，直到长度为n 个字符长度
LTRIM(str)|去掉字符串str 左侧的空格
RTRIM(str)|去掉字符串str 行尾的空格
REPEAT(str,x)|返回str 重复x 次的结果
REPLACE(str,a,b)|用字符串b 替换字符串str 中所有出现的字符串a
STRCMP(s1,s2)|比较字符串s1和s2|：比较字符串s1 和s2 的ASCII 码值的大小。如果s1 比s2 小，那么返回-1；如果s1 与s2相等，那么返回0；如果s1 比s2 大，那么返回1
TRIM(str)|去掉字符串行尾和行头的空格
SUBSTRING(str,x,y)|返回从字符串str x 位置起y 个字符长度的字串
###### 数值函数

函数 | 功能
---|---
ABS(x) | 返回x 的绝对值
CEIL(x) | 返回大于x 的最大整数值
FLOOR(x) | 返回小于x 的最大整数值
MOD(x，y) | 返回x/y 的模
RAND() | 返回0 到1 内的随机值
TRUNCATE(x,y)|返回数字x 截断为y 位小数的结果
ROUND(x,y)|返回参数x 的四舍五入的有y 位小数的值

###### 日期和时间函数

函数 |  功能
---|---
CURDATE() |返回当前日期
CURTIME() |返回当前时间
NOW() |返回当前的日期和时间
UNIX_TIMESTAMP(date) |返回日期date 的UNIX 时间戳
FROM_UNIXTIME |返回UNIX 时间戳的日期值
WEEK(date) |返回日期date 为一年中的第几周
YEAR(date) |返回日期date 的年份
HOUR(time)| 返回time 的小时值
MINUTE(time) |返回time 的分钟值
MONTHNAME(date) |返回date 的月份名
DATE_FORMAT(date,fmt) |返回按字符串fmt 格式化日期date 值
DATE_ADD(date,INTERVAL expr type) |返回一个日期或时间值加上一个时间间隔的时间值
DATEDIFF(expr,expr2) |返回起始时间expr 和结束时间expr2 之间的天数，eg：select now() current,date_add(now(),INTERVAL 31 day) after31days,date_add(now(),INTERVAL '1_2' year_month) after_oneyear_twomonth;

###### 其他函数
DATABASE() 返回当前数据库名  
VERSION() 返回当前数据库版本  
USER() 返回当前登录用户名  
MD5() 返回字符串str 的MD5 值  
show table status 查看表信息  
show table status like '%表名%'  
show index from 表名

#### MySQL 存储引擎
###### 常用命令
- 查看当前的默认存储引擎命令
```SQL
show variables like '%storage_engine%'
```
- 查看当前MySQL版本支持的引擎
```SQL
show engines;
```
- 查看某个表的存储引擎
```SQL
show create table 表名;

/**结果**/
CREATE TABLE `t1` (
  `id1` int(11) DEFAULT NULL,
  `id2` tinyint(5) unsigned DEFAULT NULL,
  `id3` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `id4` datetime DEFAULT CURRENT_TIMESTAMP,
  `id5` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
ENGINE=InnoDB即展示了表的引擎，当然由此可知，在创建的时候也可以使用这个字段指定表使用的引擎。
- 修改表的引擎

```SQL
alter table table_name engine=引擎名
```
###### 各类引擎介绍
特点 | MyISAM|InnoDB|MEMORY
---|---|---|---
存储限制|有|64TB| 有
事务安全| |支持|
锁机制 |表锁 |行锁 |表锁
B树索引|支持| 支持| 支持
哈希索引| | |支持
全文索引 |支持|
集群索引 | |支持|
数据缓存| |支持 |支持
索引缓存 |支持 |支持 |支持
数据可压缩 |支持
空间使用 |低| 高| N/A
内存使用 |低 |高| 中等
批量插入的速度| 高| 低| 高
支持外键| |支持

### 合适数据类型
###### char与varchar
- CHAR属于固定长度的字符类型，而VARCHAR属于可变长度的字符类型。由于CHAR是固定长度的，所以它的处理速度比VARCHAR快得多，但是其缺点是浪费存储空间，程序需要对行尾空格进行处理，所以对于那些长度变化不大并且对查询速度有较高要求的数据可以考虑使用CHAR类型来存储。
- 在MySQL 中，不同的存储引擎对CHAR 和VARCHAR 的使用原则有所不同，这里简单概
括如下。
1.  MyISAM 存储引擎：建议使用固定长度的数据列代替可变长度的数据列。
2.  MEMORY 存储引擎：目前都使用固定长度的数据行存储，因此无论使用CHAR 或VARCHAR 列都没有关系。两者都是作为CHAR 类型处理。
3.  InnoDB 存储引擎：建议使用VARCHAR 类型。对于InnoDB数据表，内部的行存储格式，没有区分固定长度和可变长度列（所有数据行都使用指向数据列值的头指针），因此在本质上，使用固定长度的CHAR列不一定比使用可变长度VARCHAR 列性能要好。因而，主要的性能因素是数据行使用的存储总量。由于CHAR平均占用的空间多于VARCHAR，因此使用VARCHAR来最小化需要处理的数据行的存储总量和磁盘I/O 是比较好的。
###### TEXT 与BLOB
一般在保存少量字符串的时候，我们会选择CHAR 或者VARCHAR；而在保存较大文本时，通常会选择使用TEXT 或者BLOB，二者之间的主要差别是BLOB 能用来保存二进制数据，比如照片；而TEXT 只能保存字符数据，比如一篇文章或者日记
- BLOB 和TEXT值会引起一些性能问题，特别是在执行了大量的删除操作时。删除操作会在数据表中留下很大的“空洞”，以后填入这些“空洞”的记录在插入的性能上会有影响。为了提高性能，建议定期使用OPTIMIZE TABLE功能对这类表进行碎片整理，避免因为“空洞”导致性能问题。

```SQL
// 查看表的物理内存大小
select DATA_LENGTH from information_schema.tables  where table_schema='finance-car-manage' AND table_name='t2';

// 清理零碎内存
OPTIMIZE TABLE finance-car-manage.t2;
```
- 可以使用合成的（Synthetic）索引来提高大文本字段（BLOB 或TEXT）的查询性能。简单来说，合成索引就是根据大文本字段的内容建立一个散列值，并把这个值存储在单独的数据列中，接下来就可以通过检索散列值找到数据行了。但是，要注意这种技术只能用于精确匹配的查询（散列值对于类似<或>=等范围搜索操作符是没有用处的）。可以使用MD5()函数生成散列值，也可以使用SHA1()或CRC32()，或者使用自己的应用程序逻辑来计算散列值。请记住数值型散列值可以很高效率地存储。同样，如果散列算法生成的字符串带有尾部空格，就不要把它们存储在CHAR 或VARCHAR 列中，它们会受到尾部空格去除的影响。合成的散列索引对于那些BLOB 或TEXT 数据列特别有用。用散列标识符值查找的速度比搜索BLOB 列本身的速度快很多。  
eg：
```SQL
insert into t values(1,repeat('beijing',2),md5(context));
insert into t values(2,repeat('beijing',2),md5(context));
insert into t values(3,repeat('beijing 2008',2),md5(context));
select * from t;
```
查看的内容为：

id | context |hash_value
---|--- |---
1 | beijingbeijing|09746eef633dbbccb7997dfd795cff17
2 | beijingbeijing|09746eef633dbbccb7997dfd795cff17
3 | beijing 2008beijing 2008|1c0ddb82cca9ed63e1cacbddd3f74082
- 如果要查询context 值为“beijing 2008beijing 2008”的记录，可以通过相应的散列值来
查询：
```SQL 
select * from t where hash_value=md5(repeat('beijing 2008',2));
```
###### 浮点数与定点数
- 浮点数一般用于表示含有小数部分的数值。当一个字段被定义为浮点类型后，如果插入数据的精度超过该列定义的实际精度，则插入值会被四舍五入到实际定义的精度值，然后插入，四舍五入的过程不会报错。在MySQL中float、double（或real）用来表示浮点数。
- 定点数不同于浮点数，定点数实际上是以字符串形式存放的，所以定点数可以更加精确
的保存数据。如果插入数据超过该列定义的实际精度，会报错或者报警。
- 浮点数丢失精度的其中一个原因是小数二进制十进制转换导致的。
- 注意：在今后关于浮点数和定点数的应用中，用户要考虑到以下几个原则：
1.  浮点数存在误差问题；
2.  对货币等对精度敏感的数据，应该用定点数表示或存储；
3. 在编程中，如果用到浮点数，要特别注意误差问题，并尽量避免做浮点数比较；
4. 要注意浮点数中一些特殊值的处理。