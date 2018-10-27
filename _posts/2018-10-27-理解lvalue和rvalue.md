---
layout: post
title:  理解lvalue和rvalue
date:   2018-10-27 1:05:00
catalog:  true
tags:
    - C++
    - 左值
    - 右值     
       

---

## 问题
在sublime写了一个简单的测试程序，代码如下：

```
#include <stdlib.h>
#include <stdio.h>

int main(){
	int add = &1;
	printf("%s\n%d","hello",&add);
}
```
运行后，报错如下：
`error:cannot take the address of an rvalue of type int`,
rvalue是什么呢？查了资料应该表示右值的意思。

## 左右值

下面是我对这两个单词字面的意思的猜测：

- lvalue估计来源于left value。 在赋值语句中lvalue = rvalue；位置处于左边。就是可以修改的值。
- rvalue估计来源于right value。处于赋值语句右边，是只读的不可修改的值。

接下来是我所悟到内容的详细分析：

- lvalue是可以赋值的，说明它是一个变量，它在内存中一定存在，一定有地址。所以&lvalue是有效的，能取到在内存中的地址。
访问lvalue一定会导致CPU访问存储器（相对较慢的操作）。

lvalue的例子：

```
int a;  
a = 10; // a是lvalue。   
int* p = &a; // &a是rvalue。   
&a = 0; //错误，&a不是lvalue，因为a的地址一旦分配好了，就不能改变了。 
```
- rvalue是不可以赋值的，它不是一个变量，在内存中没有存在，没有地址。它要么是存在于CPU的寄存器中，要么是存在于指令中（立即数）。所以只要对rvalue取地址，那么就一定是错误的（编译器会抱怨的）。

访问rvalue不会导致CPU访问存储器（对立即数和寄存器的访问很快）。

rvalue的例子：

```
int a;  
a = 10; // 10是rvalue，它没有地址，&10就是错误的表达式。从汇编语言的角度来看，10是直接存在于MOV指令中的立即数。   
10 = a; // 错误，10是rvalue，不可赋值。   
//函数返回值属于rvalue，因为返回值通常用CPU寄存器传递，没有地址。   
int foo()  
{  
    return 0;  
}  
int b = foo(); //没问题，函数返回值是rvalue。   
int* p = &foo(); //错误，rvalue没有地址。   
void bar(int& i)  
{  
}  
bar(foo()); //错误，bar函数参数需要的是lvalue。
```
- 函数的返回值是rvalue，对于返回int, char 等这样最基本的类型，是通过CPU寄存器返回的，因此返回值没有地址是可以理解的。但是如果函数返回的是一个用户自定义类型的对象，肯定不可能通过寄存器来返回这个对象值的（寄存器大小数量都有限，对象的大小可以非常大），那究竟是怎样返回对象的呢？
- 
```
class UDT  
{  
  int data[100];  
public:  
  UDT()  
  {  
    printf("construct/n");  
  }  
  BBB& operator = (BBB& )  
  {  
    printf("operator =/n");  
    return *this;  
  }  
};  
UDT foo()  
{  
  return UDT();  
}  
void main()  
{  
  UDT obj = foo();  
}  
//输出：   
construct  
```
带着疑问，我查了查vc编译出来的代码，原来obj这个局部变量的地址被压入了堆栈，foo函数内部以堆栈上的obj地址作为this指针调用了UDT的构造函数。噢，难怪执行UDT obj = foo();这个语句只有调用了一次构造函数，而没有调用operator =，这都是因为函数返回值必须是rvalue这个规则所带来的好处，如果返回值是一个lvalue，那么这个语句一定会调用operator = 运算符。


