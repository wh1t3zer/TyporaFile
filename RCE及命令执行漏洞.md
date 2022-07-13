# 代码执行（执行脚本代码）

## 1、脚本

1、php

2、java

3、python

## 2、产生

### 1、web源码

1、thinkphp

2、eyoucms

3、wordpress

### 2、中间件平台

1、Tomcat

2、Apache Struts2

3、redis

### 3、其他环境

1、PHP-CGI

2、Jenkins-CI

3、Java RMI

## 3、检测

### 1、白盒

代码审计

### 2、黑盒

漏扫工具

公开漏洞

手工注入

## 4、防御

### 敏感函数禁用

### 变量过滤或固定

### WAF产品

## 执行的函数

eval()

assert()

call_user_func()&call_func_array()回调函数

create_function()

# 命令执行（执行危害命令）

## 1、系统

Linux

Window

## 2、产生

### 1、web源码

Nexus

Webmin

ElasticSearch

### 2、中间件平台

Weblogic

Apache

### 3、其他环境

Postgresql

Samba

Supervisord

## 3、检测

### 1、白盒

代码审计

### 2、黑盒

漏扫工具

公开漏洞

手工注入

## 4、防御

### 敏感函数禁用

### 变量过滤或固定

### WAF产品

## 执行的函数

system()

passthru()

exec()

shell_exec()&反引号('')

注：若在eval()中加入反引号，则会被当成系统命令执行而不是代码执行

# 形成条件

### 1、可控变量

### 2、漏洞函数

