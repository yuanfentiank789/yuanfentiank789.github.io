---
layout: post
title:  python fire库的使用
date:   2019-01-18 1:05:00
catalog:  true
tags:
    - Python 
    - Fire库
         
       

---

## 介绍

fire是python中用于生成命令行界面(Command Line Interfaces, CLIs)的工具，不需要做任何额外的工作，只需要从主模块中调用fire.Fire()，它会自动将你的代码转化为CLI，Fire()的参数可以说任何的python对象

## 安装

`pip install fire
`
## 用法

### 实例1 单个函数：python 模块名 参数1 参数2

在Common目录下新建一个test_fire.py的模块

```
import fire

def add(a, b):
    count = a + b
    return count

if __name__ == '__main__':
    fire.Fire(add)
```
当不接参数时，执行python test_fire.py会显示帮助信息

![](https://images2018.cnblogs.com/blog/1186367/201809/1186367-20180902172140444-853964106.png)

帮助信息里显示了fire.Fire()的参数类型(function)、文件路径、文档字符串、参数用法等信息

加参数运行的结果如下：

注意：两种方法调用，一种是直接跟实参，一种是--形参 实参的形式，为了方便，本文采取第一种

![](https://images2018.cnblogs.com/blog/1186367/201809/1186367-20180902172435841-975530088.png)

### 实例2 多个函数：python 模块名 函数名 参数

```
import fire

def add(a, b):
    count = a + b
    return count

def sub(a, b):
    result = a - b
    return result

if __name__ == '__main__':
    fire.Fire()
```
![](https://images2018.cnblogs.com/blog/1186367/201809/1186367-20180902173106428-1736539904.png)

### 实例3 类(对象) 多个函数：python 模块名 函数名 参数

```
import fire

class Calculator(object):
    def add(self, a, b):
        count = a + b
        return count

    def sub(self, a, b):
        result = a - b
        return result

if __name__ == '__main__':
    fire.Fire(Calculator)    #这里用类名Calculator或者类的实例化对象Calculator()结果都是一样的
```
加参数运行结果：

![](https://images2018.cnblogs.com/blog/1186367/201809/1186367-20180902173806956-915514562.png)

### 注意：
1. fire 默认使用 - 作为参数分隔符，所以如果你要在命令行传入类似 2017-04-22 的参数时，那么程序接收到的参数就肯定不是 2017-04-22 了，需要使用 --separator 来改变分隔符

2. fire 会自动区分你在命令行传入的参数的类型，例如 20170422 会自动识别成 int，hello 会自动识别成 str，'(1,2)' 会自动识别成 tuple，'{"name": "Alan Lee"}' 会自动识别成 dict。但是你如果想要传入一个字符串类型的 20170422 怎么办？那就需要这样写：'"20170422"' 或者 "'20170422'" 或者 \"20170422\"，总之呢，就是加一个转义，因为命令行默认会吃掉你的引号



