---
layout: post
title:  "ViewTreeObserver引发的内存泄漏问题"
date:   2016-11-23 1:05:00
catalog:  true
tags:

   - ViewTreeObserver
   - leak
   - attachinfo
   
   
     


---

## ViewTreeObserver引发内存泄漏？
项目中业务需要监听ScrollView的滚动状态，实现如下：

addOnScrollChangedListener

    public void initView(){
    
    ....
    
    ViewTreeObserver observer = v.getViewTreeObserver();
    observer.addOnScrollChangedListener(this);
    
    ...
    
    }
    
    
removeOnScrollChangedListener

    public void removeScrollChangedListener() {
        if (null != scroll) {
            ViewTreeObserver observer = scroll.getViewTreeObserver();
            observer.removeOnScrollChangedListener(this);
        }
    }               
    
运行中发现this对象（当前页面）内存泄漏，无法被回收.明明按返回键时，调用removeScrollChangedListener方法了，为什么还是无法回收掉呢？


## 分析过程

打印log发现在addOnScrollChangedListener和removeOnScrollChangedListener时，获取到的ViewTreeObserver不是同一个实例对象，所以MinePage始终被addOnScrollChangedListener时获取到的ViewTreeObserver引用，无法回收掉。

跟踪getViewTreeObserver()代码发现源码如下：

    public ViewTreeObserver getViewTreeObserver() {
        if (mAttachInfo != null) {
            return mAttachInfo.mTreeObserver;
        }
        if (mFloatingTreeObserver == null) {
            mFloatingTreeObserver = new ViewTreeObserver();
        }
        return mFloatingTreeObserver;
    }
    
逻辑为如果mAttachInfo为null，则执行并返回一个new ViewTreeObserver()，那么mAttachInfo什么时候为空呢？mAttachInfo是一个记录view和窗口对应关系的View的内部类,那么View是否提供了判断自己是否attached到window的方法呢？查阅API文档发现，View有一个如下判断是否attached的方法：

    /**
     * Returns true if this view is currently attached to a window.
     */
    public boolean isAttachedToWindow() {
        return mAttachInfo != null;
    }

问题就出在这里，debug代码发现，我们调用addOnScrollChangedListener是在com.autonavi.mine.page.mineentry.page.MinePage#initView方法中，根据调用栈
可以知道该方法在fragment生命周期中的onCreate方法被调用，此时ScrollView的isAttachedToWindow返回值是什么呢？答案是false，也就是会执行new ViewTreeObserver()，返回一个和attachinfo无关的ViewTreeObserver，这一点可以通过打印日志验证。

那么removeScrollChangedListener被调用时，ScrollView的isAttachedToWindow返回值是什么呢？答案是true，此时获取到的是mAttachInfo.mTreeObserver这个observer。
mAttachInfo实例是窗口级别共享的，viewroot会递归传递给自己的子view。

## 解决方案


监听view的attach状态，在attach时再调用addOnScrollChangedListener，detach时调用removeScrollChangedListener，如下：

    if (null != scroll) {
            scroll.addOnAttachStateChangeListener(new View.OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    ViewTreeObserver observer = v.getViewTreeObserver();
                    observer.addOnScrollChangedListener(MinePage.this);//attach时注册
                }

                @Override
                public void onViewDetachedFromWindow(View v) {
                    removeScrollChangedListener();//detach时反注册
                }
            });
        }
        
 打印log可发现，removeScrollChangedListener和addOnScrollChangedListener时获取到的observer为同一个实例，this引用可以被释放掉了。
 

 
## 更多

观察ViewTreeObserver的API，发现还有大量addListener相关的方法，

    public void addOnDrawListener(OnDrawListener listener)
    
    public void addOnEnterAnimationCompleteListener(OnEnterAnimationCompleteListener listener)
    
    public void addOnGlobalFocusChangeListener(OnGlobalFocusChangeListener listener)
    
    public void addOnGlobalLayoutListener(OnGlobalLayoutListener listener)
    
        
使用时都应该注意到会引发这个内存泄漏问题。