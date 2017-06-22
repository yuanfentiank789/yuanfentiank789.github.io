---

layout: post
title:  "跳出手机的Dialog---Presentation"
date:   2017-06-21 1:05:00
catalog:  true
tags:

   - Presentation
   - 分屏
   
   
       
   
---

[转载自http://www.th7.cn/Program/Android/201607/901164.shtml](http://www.th7.cn/Program/Android/201607/901164.shtml)

<div class="article">
       

        <!-- 作者区域 -->
        <div class="author">
          <a class="avatar" href="/u/f5909165c1e8">
           
</a>          <div class="info">
                        <!-- 文章数据信息 -->
                      </div>
          <!-- 如果是当前作者，加入编辑按钮 -->
        </div>
        <!-- -->

        <!-- 文章内容 -->
        <div class="show-content">
          <h1>跳出手机的Dialog--Presentation</h1>
<blockquote><p>本文原创，转载请经过本人准许。</p></blockquote>
<p><strong>写在前面：</strong></p>
<p><strong>Presentation 是 what？</strong><br>也许你刚看到标题的时候，会默默把这个单词扔到翻译工具里面，就像老大最开始跟我提起这个单词的时候一样。</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-7e831736d001e10a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-7e831736d001e10a?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">容我想想</div>
</div>
<p>Presentation是说明书？<br>Presentation是一个颁奖典礼？<br><strong>Presentation还是某卖药公司UE总监让所有IT人尴尬癌尽犯的PPT Presentation？</strong></p>
<p>公司做的是智能硬件方向，国内有关Presentation资料几乎是空白的，所以我的研究更多参考了Presentation官方文档和一些英文资料。<strong>如果各位看官有什么建议，一定要记得补充。</strong></p>
<h2>presentation的定义</h2>
<p>好吧，既然上面这些都不是，Presentation是一个类，我们翻过太平洋的墙，来看看Presentation的定义。</p>
<blockquote><p>A presentation is a special kind of dialog whose purpose is to present content on a secondary display.</p></blockquote>
<p>我们仅仅先来看这一句定义，因为当你对一个东西完全不了解时候，知道的越多，越会影响你的判断。</p>
<p>翻译下：presentation 是一种特殊的 <strong>dialog</strong> ，目的是为了在<strong>辅助屏幕</strong>上展示不同的内容。</p>
<p>在这句话上，我收集到了两个关键的信息：</p>
<ul>
<li>
<p><strong>presentation 是一个 dialog</strong><br>根据<strong>生物遗传学</strong>的角度，presentation 无论被描述成什么天花乱坠的模样，它也是一个dialog。</p>
</li>
<li>
<p><strong>presentation 目的是显示在辅助屏幕上</strong></p>
</li>
</ul>
<p>进一步思考下，也就是说：</p>
<p>我可以拿着我自己的手机，点击一个按钮，然后在你的电脑上或者手机上，弹出一个<strong>自定义的Dialog</strong>（<strong>脑补一下恶作剧场景O(∩_∩)O</strong>）？</p>
<p>这与我们之前，通过一些软件，将手机屏幕同步到电脑上，<strong>区别</strong>又在哪里呢？</p>
<p>相信很多人都能立刻想明白，区别在于：<strong>展示不同内容</strong></p>
<p>通过软件同步到电脑，展示的东西始终与我的手机屏幕相同。</p>
<p>而利用 presentation 我可以自由的展示我想展示的内容，因为它是一个Dialog，是局部可控的。</p>
<h2>寻找并投影到辅助屏幕</h2>
<p>产品经理找到我，向我提出了以下几个疑问：</p>
<p>现在手里有一部Android手机</p>
<p> 能否连接以下几种设备</p>
<ul>
<li>
<p>另一部Android手机</p>
</li>
<li>
<p>笔记本电脑</p>
</li>
<li>
<p>智能电视</p>
</li>
<li>
<p>小米盒子等</p>
</li>
</ul>
<p>并且连接之后，利用presentation展示不同内容。</p>
<p>我乍一看这几个设备，感觉都没问题呀。可是当我拿着手机挨个尝试，几次失败，并且耐心分析之后，发现了问题。</p>
<p>首先Presentation是Android 4.2引出的，与之同时Android 4.2 还支持 Miracast 影像传输协议。所以它俩一定是有联系的。</p>
<p><strong>Miracast</strong> </p>
<p><a href="http://www.360doc.com/content/15/0422/21/1204156_465287702.shtml" target="_blank">Miracast简介</a></p>
<p>Miracast是一种基于WIFI的传输协议，Android 4.2以上的手机、Win8电脑、智能电视、盒子几乎都是支持它的。</p>
<p>不过Miracast它将设备分为发送端和接收端<br>发送端有手机、电脑。<br>接收端有智能电视、电视盒子。</p>
<p>所以，手机连手机或电脑展示Presentation，是行不通的。手机作为发射端，去寻找智能电视和盒子才是正解。</p>
<h2>Presentation</h2>
<p>终于弄明白了要寻找的设备是怎样的，建立连接之前，参考官方文档的样例，我们先把<strong>Presentation</strong>给搭建好。</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-9b65f51ad6fddc47?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-9b65f51ad6fddc47?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">Presentation类</div>
</div>
<p>可以看到，和Activity一样，可以通过setContentView来给Presentation设置一个布局。自然布局里可以有各种各样的组件，还可以有像GLSurfaceView、SurfaceView 这种重量级的组件，来显示炫酷的动画。这里我们就仅仅写一个TextView，展示一行“show a Presentation”文字。</p>
<p>值得一提的是，在Presentation中的getContext得到的context与它依附的Activity的context是不同的，Presentation的context是目标屏幕属性的context，包含着辅助屏幕的属性信息。</p>
<h2>获取辅助屏幕</h2>
<p>获取辅助屏幕有两种方式</p>
<ul>
<li>
<p><strong>MediaRouter</strong></p>
</li>
<li>
<p><strong>DisplayManager</strong></p>
</li>
</ul>
<p><strong>MediaRouter</strong></p>
<p>利用MediaRouter的API寻找周围设备是一种最简单的方式了，它会直接绑定周围最合适的设备。就相当于你用谷歌搜索直接点击“<strong>手气不错</strong>”</p>
<p>代码如下：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-a732a40ac5b0917c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-a732a40ac5b0917c?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">点击按钮，展示Presentation</div>
</div>
<p>可以看到在Presentation的构造中，传入了一个display，这就是搜索到的那个设备</p>
<p>先来测试一下，Android 4.2的手机在开发中选项中，都有模拟辅助屏幕的功能，我们选择一个分辨率，打开它，模拟一个外部的屏幕。</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-ac89976cb382208d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-ac89976cb382208d?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">打开模拟辅助屏幕</div>
</div>
<p>默认辅助屏幕是同步手机屏幕的，打开之后，进入测试app，点击按钮：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-bf37642c0f80de9e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-bf37642c0f80de9e?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">这里写图片描述</div>
</div>
<p>注意这可不是一个Dialog，而是我们把内容展示在了一个模拟的辅助屏幕上，回头看看标题，是不是就实现了呢？</p>
<p><strong>DisplayManager</strong></p>
<p>第二种搜索设备的方法是DisplayManager，他可以搜索周围所有可用的display，产生一个display数组，然后你就可以选择合适的设备进行展示了。</p>
<p>代码如下：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-64ae611bb26eb676?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-64ae611bb26eb676?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">DisplayManager</div>
</div>
<p>代码还是挺简单的，搜索到周围所有可用设备之后，展示到ListView上，点击条目，在APP上和Presentataion上分别跑一个秒表，看看延时性如何，截图如下。</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/1915184-3eaea827329729f0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/1915184-3eaea827329729f0?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">DisplayManager</div>
</div>
<p>可以看到，搜索到的设备名称是  <strong>叠加视图#1</strong> ，点击条目之后两个秒表也分别跑了起来。</p>
<p><strong>总结：</strong></p>
<p>上面对Presentation进行了一个简略的介绍，因为相信大家如果做的不是智能硬件方向，基本上不会遇到这个需求。关于Activity对Presentation的管理方式，官方文档的有两个Demo可以参考，需要时可以去查看。</p>
<p><strong>写在后面：</strong></p>
<p>周末在连接智能电视测试时，发现延时很小，完全可以投入使用。<br>关于Presentation资料比较少，欢迎大家一同交流</p>

        </div>
        <!--  -->

            </div>