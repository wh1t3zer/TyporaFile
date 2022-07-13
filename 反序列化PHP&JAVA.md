# 1、原理

序列化就是将对象转换成字符串，反序列化相反，数据的格式的转换对象的序列化利于对象的保存和传输，也可以让多个文件共享对象

java、php、js、c#通过序列化转换成二进制、XML、json等其他字符

# 2、技术

## 有类

触发魔术方法

1、__construct

2、__destruct

3、__wakeup

4、__toString

.....

## 无类

# 3、利用

真实应用

CTF比赛

# 4、危害

SQL注入

代码执行

目录遍历

...





# PHP反序列化

原理：未对用户输入的序列化字符串进行检测，导致攻击者可以控制反序列化进程，从而导致代码执行。

serialize()   //将一个对象转换成一个字符串

unserialize()   //将字符串还原成一个对象



触发unserialize函数的变量可控，文件中存在可利用的类，类中有魔术方法

__construct() //创建对象时触发

__destruct()  //对象被销毁时触发

__call()      //在对象上下文中调用不可访问的方法时触发

__callStatic()  //在静态上下文中调用不可访问的方法时触发

__get()      //用于从不可访问的属性读取数据

__isset()   //在不可访问的属性上调用isset()或empty()触发

__unset()  //在不可访问的属性上使用unset()触发

__invoke() //当脚本尝试将对象调用为函数时触发

__toString() //当有echo或者拼接字符串或隐性调用该方法都会被触发

https://www.cnblogs.com/20175211lyz/p/11403397.html



调用serialize()会自动调用sleep()魔术方法，调用unserialize()会自动调用wakeup()魔术方法 

# JAVA反序列化

ObjectOutputStream类  ----> writeObject()  序列化

ObjectInputStream类  ----> readObject()     反序列化

## 1、利用

1、Payload生成器    ysoserial

2、自定义检测工具或脚本

## 2、检测

### 黑盒

#### 一、数据格式点

1、HTTP请求中的参数

2、自定义协议

3、RMI协议

4、子主题5

....

#### 二、特定扫描

### 白盒

#### 一、函数点

ObjectInputStream.readObject

ObjectInputStream.readUnshared

XMLDecoder.readObject

XStream.fromXML

ObjectMapper.readValue

JSON.parseObject

...

#### 二、组件点

ysoserial库

https://github.com/frohoff/ysoserial

#### 三、代码点

RCE执行

数据认证

....

下方特征可作为序列化的标志参考

一段数据以rO0AB开头，基本可以确定这串就是JAVA序列化base64加密的数据

如果以aced开头，那么它就是这一段JAVA序列化的16进制