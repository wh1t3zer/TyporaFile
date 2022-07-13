# WAF分类

1、云waf在配置云waf时（通常是CDN包含的waf），DNS需要解析到CDN的ip上去，在请求uri时，数据包就会先经过云waf进行检测，如果通过再将数据包流给主机。

2、主机防护软件在主机上预先安装了这种防护软件，可用于扫描和保护主机，和监听web端口的流量是否有恶意的，所以这种从功能上讲较为全面。这里再插一嘴，mod_security、ngx-lua-waf这类开源waf虽然看起来不错，但是有个弱点就是升级的成本会高一些。

3、硬件ips/ids防护、硬件waf使用专门硬件防护设备的方式，当向主机请求时，会先将流量经过此设备进行流量清洗和拦截，如果通过再将数据包流给主机。

# WAF身份认证阶段的绕过

## 伪造搜索引擎

早些版本的安全狗是有这个漏洞的，就是把User-Agent修改为搜索引擎，便可以绕过，进行sql注入等攻击，这里推荐一个谷歌插件，可以修改User-Agent，叫User-Agent Switcher。

## 伪造白名单特殊目录

360webscan脚本存在这个问题，就是判断是否为admin dede install等目录，如果是则不做拦截,比如GET /pen/news.php?id=1 union select user,password from mysql.user可以改为GET /pen/news.php/admin?id=1 union select user,password from mysql.user或者GET /pen/admin/…\news.php?id=1 union select user,password from mysql.user

## 直接攻击源站：

这个方法可以用于安全宝、加速乐等云WAF，云WAF的原理通过DNS解析到云WAF，访问网站的流量要经过指定的DNS服务器解析，然后进入WAF节点进行过滤，最后访问原始服务器，如果我们能通过一些手段（比如c段、社工）找到原始的服务器地址，便可以绕过。



# WAF数据包解析阶段的绕过

## 编码绕过

最常见的方法之一，比如：url encode。

## 修改请求方式绕过

大家都知道cookie中转注入，最典型的修改请求方式绕过，很多的asp，aspx网站都存在这个问题，有时候WAF对GET进行了过滤，但是Cookie甚至POST参数却没有检测。还有就是参数污染，典型例子就是multipart请求绕过，在POST请求中添加一个上传文件，绕过了绝大多数WAF。

## 复参数绕过

例如一个请求是这样的GET /pen/news.php?id=1 union select user,password from MySQL.user可以修改为GET /pen/news.php?id=1&id=union&id=select&id=user,password&id=from%20mysql.user很多WAF都可以这样绕。

# WAF触发规则的绕过

特殊字符替换空格用一些特殊字符代替空格，比如在mysql中%0a是换行，可以代替空格，这个方法也可以部分绕过最新版本的安全狗，在sqlserver中可以用//代替空格。
特殊字符拼接把特殊字符拼接起来绕过WAF的检测，比如在Mysql中，可以利用注释//来绕过，在mssql中，函数里面可以用+来拼接,例如GET /pen/news.php?id=1;exec(master…xp_cmdshell ‘net user’)可以改为GET /pen/news.php?id=1; exec(‘maste’+‘r…xp’+’