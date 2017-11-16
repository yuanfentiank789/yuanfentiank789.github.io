---

layout: post
title:  "Android中Enum的替代方案"
date:   2017-11-16 1:06:00
catalog:  true
tags:

   - intdef
   
   
   
       
   
---


在Android的官方文档的内存管理中http://developer.android.com/training/articles/memory.html#Overhead 提到由于Enum通常需要两倍以上的存储空间，因此应当尽量避免Enum的使用。对此，Google官方推出了IntDef作为对Enum的替代。以下通过一个具体的例子来说明其用法：
例如我们有如下的一组Enum:

```
public enum Color{  
RED, BLUE, YELLOW, GREEN, PURPLE, WHITE, BLACK  
}  
```

现在我们要做的是找出其替代方法，首先我们要做的是定义常量：

```
public static final int RED = 0;  
public static final int BLUE = 1;  
public static final int YELLOW = 2;  
public static final int GREEN = 3;  
public static final int PURPLE = 4;  
public static final int WHITE = 5;  
public static final int BLACK = 6;  
```
在常量定义了之后，事实上我们已经可以使之作为对Enum的替代了，但是在实际的开发过程中写的代码如果换成了其他的变量名，编译器并不能够报错。基于此背景，IntDef应运而生。在定义了常量之后，我们首先需要用一个@IntDef({})将其全部变量包含，其次需要一个Retention声明其保留级别，最后定义其接口名称，具体代码为：

```
@IntDef({RED, BLUE, YELLOW, GREEN, PURPLE, WHITE, BLACK})  
@Retention(RetentionPolicy.SOURCE)  
public @interface Color{};  
```
在使用的时候，例如我们有一个变量名称为：

```
int color;  
```
与此同时有一个函数：

```
void setColor(@Color int COLOR){  
color = COLOR;  
}  

```
在调用此函数的时候，参数名称如果不是IntDef中的变量名称的时候，例如setColor(2)，Android Studio中就会提示错误（虽然编译仍然会通过）。
在使用的时候需要在gradle中加入：

```
compile 'com.android.support:support-annotations:23.0.1'  
```


