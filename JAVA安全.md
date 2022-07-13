# 一、综合常规

## 1、SQL注入

## 2、路径遍历

上传文件可在输入框传入../../等进行上传路径的改变

## 3、XSS跨站

## 4、反序列化

## 5、XML&XXE

## 6、CSRF&SSRF

# 二、访问控制

## 1、对象引用

## 2、缺少功能

# 三、身份验证

## 1、身份验证绕过

## 2、JWT令牌

## 3、重设密码

## 4、安全密码

# 四、客户端安全

## 1、前端限制

## 2、客户端过滤

## 3、HTML篡改

# 五、组件安全



# 预防SQL注入

1、采用session防御，session内容正常情况下用户是无法修改的

select * from users where user ="'" + session.getAttribute("UserID") + "'";

2、参数绑定方式，利用SQL的预编译技术（只能预防大部分，不是能够绝对的进行sql注入防御） 

https://www.cnblogs.com/klyjb/p/11473857.html

注入语句中的union，select不再是SQL编译语句，而是一个参数

通过 case when语句可以将order by后的orderExpression表达式中添加select语句

## 预编译绕过

# JWT安全（JSON Web Token）

分为三部分，头部，声明，签名，每个部分以英文句号.隔开。内容以BASE64URL进行了编码

## 一、解释

JSON Web Token是一种跨域验证身份的方案。JWT不加密传输的数据，但能够通过数字签名来验证数据未被篡改

## 二、组成

头部（Header）

​	alg   常见值用HS256（默认），HS512等，也可以为None。HS256表示HMAC SHA256

​	typ  说明这个token类型为JWT

声明（Claims）

​	iss：发行人 

​	exp：到期时间 

​	sub：主题

​	aud：用户

​	nbf：在此之前不可用

​	iat：发布时间

​	jti：JWT ID用于标识该JWT

签名（Signature）

## 三、攻击

1、伪造

​		无密钥-修改alg，删除签名（none)

2、爆破

​		针对签名密钥爆破

3、配合

​		JWT数据中存在参数传递接受处理过程

## 四、检测

1、javaweb

2、Authorization

3、数据包数据格式