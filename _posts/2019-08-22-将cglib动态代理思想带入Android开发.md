---
layout: post
title:  将cglib动态代理思想带入Android开发
date:   2019-08-22 1:05:00
catalog:  true
tags:
    - 动态代理
    - cglib
    
         
       

---

<div class="show-content-free">
            <p>动态代理在Android实际开发中用的并不是很多，但在设计框架的时候用的就比较多了，最近在看J2EE一些东西，像Spring，Hibernate等都有通过动态代理来实现方法增强、方法拦截等需要，通过代理的方式优雅的实现AOP编程。我们今天来看看这个代理究竟是什么样子，在Android开发中如何使用它，以及将cglib动态代理思想在Android中看看如何实现。</p>
<p><strong>项目地址：<a href="https://link.jianshu.com?t=https://github.com/zhangke3016/MethodInterceptProxy" target="_blank" rel="nofollow">MethodInterceptProxy</a></strong></p>
<h3>一、什么是代理</h3>
<p>通常我们说的代理，在生活中就像中介、经纪人的角色。</p>
<p>目标对象/被代理对象 ------ 房主：真正的租房的方法<br>
代理对象 ------- 黑中介：有租房子的方法（调用房主的租房的方法）<br>
执行代理对象方法的对象 ---- 租房的人</p>
<p>流程：我们要租房-----&gt;中介（租房的方法）------&gt;房主（租房的方法）<br>
抽象：调用对象-----&gt;代理对象------&gt;目标对象</p>
<h3>二、静态代理</h3>
<p>先看看比较常见的静态代理，也就是装饰设计模式：<br>
首先建一个Star接口：</p>
<pre class="hljs java"><code class="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Star</span> </span>{

    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">singSong</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p>然后建Star子类SuperStar</p>
<pre class="hljs java"><code class="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SuperStar</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Star</span> </span>{

    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">singSong</span><span class="hljs-params">()</span> </span>{
        System.out.println(<span class="hljs-string">"唱歌啦--------"</span>);
    }

}
</code></pre>
<p>最后我们创建SuperStar的代理类SuperStarProxy</p>
<pre class="hljs java"><code class="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SuperStarProxy</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Star</span> </span>{

    <span class="hljs-keyword">private</span> Star star;
    
    SuperStarProxy(Star star){
        <span class="hljs-keyword">this</span>.star = star;
    }
    
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">singSong</span><span class="hljs-params">()</span> </span>{
        System.out.println(<span class="hljs-string">"before-------------"</span>);
        star.singSong();
        System.out.println(<span class="hljs-string">"after-------------"</span>);
    }
}

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
        Star star = <span class="hljs-keyword">new</span> SuperStarProxy(<span class="hljs-keyword">new</span> SuperStar());
        star.singSong();
    }
</code></pre>
<p>我们将需要代理的对象传进来生成代理对象，之后只需要使用代理对象来处理相关业务就可以了。</p>
<h3>三、动态代理</h3>
<p>静态代理需要为每一个需要代理的类写一个代理类，为每一个需要代理的方法重写代理方法，如果有上百个类或者类里方法很多，那重复的工作量也是很可观的。JDK提供了动态代理方式，可以简单理解为在JVM可以在运行时帮我们动态生成一系列的代理类，这样我们就不需要手写每一个静态的代理类了。</p>
<pre class="hljs java"><code class="java">        <span class="hljs-keyword">final</span> Star star = <span class="hljs-keyword">new</span> SuperStar();
        Star st = (Star) Proxy.newProxyInstance(Star.class.getClassLoader(), star.getClass().getInterfaces(), <span class="hljs-keyword">new</span> InvocationHandler() {
            
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">invoke</span><span class="hljs-params">(Object proxy, Method method, Object[] args)</span>
                    <span class="hljs-keyword">throws</span> Throwable </span>{
                System.out.println(<span class="hljs-string">"before---------"</span>);
                Object object = method.invoke(star, args);
                System.out.println(<span class="hljs-string">"after---------"</span>);

                <span class="hljs-keyword">return</span> object;
            }
        });
        st.singSong();
</code></pre>
<blockquote>
<p>newProxyInstance(ClassLoader loader, Class&lt;?&gt;[] interfaces, InvocationHandler h)</p>
</blockquote>
<blockquote>
<p>参数一：类加载器，动态代理类，运行时创建。一般情况：当前类.class.getClassLoader()<br>
参数二：interfaces：代表与目标对象实现的所有的接口字节码对象数组<br>
参数三：具体的代理的操作，InvocationHandler接口</p>
</blockquote>
<p>通过JDK提供的动态代理方式，我们可以很轻松的生成代理对象，但通过这种方式实现代理有个很大的限制就是：<strong>JDK的Proxy方式实现的动态代理，目标对象必须有接口，没有接口不能实现jdk版动态代理。</strong></p>
<h3>四、cglib</h3>
<p>cglib是一个功能强大，高性能的代码生成包。它为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。通常可以使用Java的动态代理创建代理，但当要代理的类没有实现接口或者为了更好的性能，cglib是一个好的选择。<br>
但是但是但是，一个很致命的缺点是：cglib底层采用ASM字节码生成框架，使用字节码技术生成代理类，也就是生成的.class文件，而我们在android中加载的是优化后的.dex文件，也就是说我们需要可以动态生成.dex文件代理类，cglib在android中是不能使用的。但后面我们会根据dexmaker框架来仿照动态生成.dex文件，实现cglib的动态代理功能。<br>
好了，我们先来看下cglib的强大吧～<br>
举个例子，boss安排要实现一个人员管理的增删改查功能，那这个简单，三两下就搞定：</p>
<pre class="hljs cpp"><code class="cpp"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PeopleService</span> {</span>

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">add</span><span class="hljs-params">()</span></span>{
        System.out.println(<span class="hljs-string">"add-----------"</span>);
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">()</span></span>{
        System.out.println(<span class="hljs-string">"delete-----------"</span>);
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">update</span><span class="hljs-params">()</span></span>{
        System.out.println(<span class="hljs-string">"update-----------"</span>);
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">select</span><span class="hljs-params">()</span></span>{
        System.out.println(<span class="hljs-string">"select-----------"</span>);
    }
}
</code></pre>
<p>OK,搞定～但是呢，需求是不断变化的，过了几天，boss又发话了，说不是每个人都可以使用这个增删改查功能的，要指定人员才可以使用，那。。我们就改吧～最直接的方式，在每个方法上都加上判断条件，但这么做多少有点太挫了，如果有五十个方法，那我们这么多方法就要都加一遍，用cglib我们可以这么做：</p>
<pre class="hljs java"><code class="java">        <span class="hljs-keyword">final</span> String name = <span class="hljs-string">"张si"</span>;
        Enhancer enhancer = <span class="hljs-keyword">new</span> Enhancer();
        enhancer.setSuperclass(PeopleService.class);
        <span class="hljs-comment">//目标对象拦截器，实现MethodInterceptor </span>
        <span class="hljs-comment">//Object object为目标对象 </span>
        <span class="hljs-comment">//Method method为目标方法 </span>
        <span class="hljs-comment">//Object[] args 为参数， </span>
        <span class="hljs-comment">//MethodProxy proxy CGlib方法代理对象 </span>
        enhancer.setCallback(<span class="hljs-keyword">new</span> MethodInterceptor() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">intercept</span><span class="hljs-params">(Object object, Method method, Object[] args,
                    MethodProxy proxy)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
                Object obj = <span class="hljs-keyword">null</span>;
                <span class="hljs-keyword">if</span>(name.equals(<span class="hljs-string">"张三"</span>)){
                     obj = proxy.invokeSuper(object, args); ;
                }<span class="hljs-keyword">else</span>{
                    System.out.println(<span class="hljs-string">"----对不起，您没有权限----"</span>);
                }
                <span class="hljs-keyword">return</span> obj;
            }
        });
        PeopleService ps = (PeopleService) enhancer.create();
        ps.add();
</code></pre>
<p>只需添加一个拦截器，就可以拦截所有增删改查操作，从切面进行统一管理，代码量也不多。<br>
又过了几天，boss又发话了，我们的增删改查不是都要有限制，只有查找才对特定人员有限制，那我们就继续改喽～<br>
这个时候我们就可以加入过滤器CallbackFilter：</p>
<pre class="hljs java"><code class="java"><span class="hljs-keyword">final</span> String name = <span class="hljs-string">"张武"</span>;
        Enhancer enhancer = <span class="hljs-keyword">new</span> Enhancer();
        enhancer.setSuperclass(PeopleService.class);
        <span class="hljs-comment">//目标对象拦截器，实现MethodInterceptor </span>
        <span class="hljs-comment">//Object object为目标对象 </span>
        <span class="hljs-comment">//Method method为目标方法 </span>
        <span class="hljs-comment">//Object[] args 为参数， </span>
        <span class="hljs-comment">//MethodProxy proxy CGlib方法代理对象 </span>
        MethodInterceptor interceptor = <span class="hljs-keyword">new</span> MethodInterceptor() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">intercept</span><span class="hljs-params">(Object object, Method method, Object[] args,
                    MethodProxy proxy)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
                Object obj = <span class="hljs-keyword">null</span>;
                <span class="hljs-keyword">if</span>(name.equals(<span class="hljs-string">"张三"</span>)){
                     obj = proxy.invokeSuper(object, args); ;
                }<span class="hljs-keyword">else</span>{
                    System.out.println(<span class="hljs-string">"----对不起，您没有权限----"</span>);
                }
                <span class="hljs-keyword">return</span> obj;
            }
        };
        <span class="hljs-comment">//NoOp.INSTANCE：这个NoOp表示no operator，即什么操作也不做，代理类直接调用被代理的方法不进行拦截</span>
        enhancer.setCallbacks(<span class="hljs-keyword">new</span> Callback[]{interceptor,NoOp.INSTANCE});
        enhancer.setCallbackFilter(<span class="hljs-keyword">new</span> CallbackFilter() {
            <span class="hljs-comment">//过滤方法 </span>
            <span class="hljs-comment">//返回的值为数字，代表了Callback数组中的索引位置，要到用的Callback </span>
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">accept</span><span class="hljs-params">(Method method)</span> </span>{
                <span class="hljs-keyword">if</span>(method.getName().equals(<span class="hljs-string">"select"</span>)){
                    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
                }
                <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>;
            }
        });
        PeopleService ps = (PeopleService) enhancer.create();
        ps.add();
</code></pre>
<p>我们没有修改一行原有的增删改查代码，通过传递代理对象的方式轻松解决方法拦截、方法增强的业务需求。<br>
但遗憾的是，cglib不支持android平台。。。那我们就自己实现咯～在dexmaker和cglib-for-android库的基础上，修改部分代码后形成我们的类似cglib框架 <strong><a href="https://link.jianshu.com?t=https://github.com/zhangke3016/MethodInterceptProxy" target="_blank" rel="nofollow">MethodInterceptProxy</a></strong> ，实现上面需求只需这样写，和cglib写法一致:</p>
<pre class="hljs java"><code class="java">        <span class="hljs-keyword">final</span> String name = <span class="hljs-string">"张五"</span>;
        Enhancer enhancer = <span class="hljs-keyword">new</span> Enhancer();
        enhancer.setSuperclass(PeopleService.class);
        <span class="hljs-comment">//目标对象拦截器，实现MethodInterceptor </span>
        <span class="hljs-comment">//Object object为目标对象 </span>
        <span class="hljs-comment">//Method method为目标方法 </span>
        <span class="hljs-comment">//Object[] args 为参数， </span>
        <span class="hljs-comment">//MethodProxy proxy CGlib方法代理对象 </span>
        MethodInterceptor interceptor = <span class="hljs-keyword">new</span> MethodInterceptor() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">intercept</span><span class="hljs-params">(Object object, Object[] args, MethodProxy methodProxy)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
                Object obj = <span class="hljs-keyword">null</span>;
                <span class="hljs-keyword">if</span>(name.equals(<span class="hljs-string">"张三"</span>)){
                     obj = methodProxy.invokeSuper(object, args); ;
                }<span class="hljs-keyword">else</span>{
                    System.out.println(<span class="hljs-string">"----对不起，您没有权限----"</span>);
                }
                <span class="hljs-keyword">return</span> obj;
            }
        };
        <span class="hljs-comment">//NoOp.INSTANCE：这个NoOp表示no operator，即什么操作也不做，代理类直接调用被代理的方法不进行拦截</span>
        enhancer.setCallbacks(<span class="hljs-keyword">new</span> MethodInterceptor[]{interceptor,NoOp.INSTANCE});
        enhancer.setCallbackFilter(<span class="hljs-keyword">new</span> CallbackFilter() {
            <span class="hljs-comment">//过滤方法 </span>
            <span class="hljs-comment">//返回的值为数字，代表了Callback数组中的索引位置，要到用的Callback </span>
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">accept</span><span class="hljs-params">(Method method)</span> </span>{
                <span class="hljs-keyword">if</span>(method.getName().equals(<span class="hljs-string">"select"</span>)){
                    <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
                }
                <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>;
            }
        });
        PeopleService ps = (PeopleService) enhancer.create();
        ps.add();
</code></pre>
<p><strong>项目地址：<a href="https://link.jianshu.com?t=https://github.com/zhangke3016/MethodInterceptProxy" target="_blank" rel="nofollow">MethodInterceptProxy</a></strong></p>

          </div>


