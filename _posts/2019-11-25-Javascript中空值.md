---
layout: post
title:  Javascript中的undefined、null、""、0值和false的区别总结
date:   2019-11-25 1:05:00
catalog:  true
tags:
    - JavaScript 
             
       

---

在程序语言中定义的各种各样的数据类型中，我们都会为其定义一个"空值"或"假值"，比如对象类型的空值null，.NET Framework中数据库字段的空值DBNull，boolean类型的假值false等等。在JavaScript中也有很多种的"空值"和"假值"，那么它们都有什么共同点和不同点呢？ 

其实标题里面我已经列出了JavaScript中所有的"空值"和"假值"，除了boolean值本身就是true和false这两种情况外，其它数据类型的"空值"主要是undefined和defined这两大类。这些空值的类型分别是： 

```
typeof(undefined) == 'undefined' 
typeof(null) == 'object' 
typeof("") == 'string' 
typeof(0) == 'number' 
typeof(false) == 'boolean' 
```
这五个值的共同点是，在if语句中做判断，都会执行false分支。当然从广义上来看，是说明这些数值都是其对应数据类型上的无效值或空值。还有这五个值作!运算，结果全为：true。 

这几个值中也有不同，其中undefined和null比较特殊，虽然null的类型是object，但是null不具有任何对象的特性，就是说我们并不能执行null.toString()、null.constructor等对象实例的默认调用。所以从这个意义上来说，null和undefined有最大的相似性。看看null == undefined的结果(true)也就更加能说明这点。不过相似归相似，还是有区别的，就是和数字运算时，10 + null结果为：10；10 + undefined结果为：NaN。 

另外""、0和false虽然在if语句表现为"假值"，可它们都是有意义数据，只是被作为了"空值"或"假值"，因为："".toString()，(0).toString()和false.toString()都是合法的可执行表达式。 

其实这5个值在上面所说的这些差异里，并不太会给程流程控制带来太大的问题，那么要区分它们什么呢？需要注意区分的是这些值在转换为String时的差异是比较大的，它们到String的转换关系是：

```
String(undefined) //-> "undefined" 
String(null) //-> "null" 
String("") //-> "" 
String(0) //-> "0" 
String(false) //-> "false"
```


