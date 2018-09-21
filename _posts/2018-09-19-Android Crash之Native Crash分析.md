---
layout: post
title:  Android Crash之Native Crash分析
date:   2018-09-16 1:05:00
catalog:  true
tags:
    - native crash
       
       

---

<div class="markdown_views">
							<!-- flowchart 箭头图标 勿删 -->
							<svg xmlns="http://www.w3.org/2000/svg" style="display: none;"><path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path></svg>
							<h1 id="前言"><a name="t0"></a>前言</h1>

<p>上一篇给大家介绍了<a href="http://blog.csdn.net/wwj_748/article/details/51520020" rel="nofollow" target="_blank">Android Crash中的Java Crash分析</a>，我们可以知道Java Crash一般会弹出提示框告诉我们程序崩溃了，通常使用Crash工具都能够捕获到；本篇博客来谈谈如何针对Native Crash进行分析，它相对与Java层面的Crash有什么特点？如何判断程序Crash是因为Native层导致的？我们怎么去分析它？下面我们一个一个解答这些问题。</p>



<h2 id="native-crash在android上的特点"><a name="t1"></a>Native Crash在Android上的特点</h2>

<ol>
<li>出错时界面不会弹出提示框提醒程序崩溃（Android 5.0以下）</li>
<li>出错时会弹出提示框提醒程序崩溃（Android 5.0以上）</li>
<li>程序会直接闪退到系统桌面</li>
<li>这类错误一般是由C++层代码错误引起的</li>
<li>绝大部分Crash工具不能够捕获</li>
</ol>

<p>我们在实际Android开发的时候，可能会引入第三方的一些so库或者自己开发相应的so库供程序使用，然而so库一般是通过<strong>c或者c++</strong>开发的。Android开发者通过java层的JNI机制调用Native语言写的函数，然而Natice语言也可以调用java层的函数。 如果有同学不明白的话，建议先去了解下JNI的相应技术，总的来说通过JNI技术，就让我们让Java世界跟Native世界可以联系在一起，也因为这个特性，让Java具有跨平台的特性。</p>

<blockquote>
  <p>如果想了解如何通过Android Studio制作so库，笔者前面的一篇文章可以帮到你：<a href="http://blog.csdn.net/wwj_748/article/details/51274580" rel="nofollow" target="_blank">http://blog.csdn.net/wwj_748/article/details/51274580</a></p>
</blockquote>

<h2 id="native-crash是如何产生的"><a name="t2"></a>Native Crash是如何产生的？</h2>

<p>上一节我们谈到so库是同通过Native语言开发的，自然在Android中使用so库的时候发生的Crash，就是我们所说的Native Crash。为了更好的让大家知道Native Crash是如何产生的，下面笔者举一个例子：</p>

<p><strong>Java层定义Native方法</strong></p>

<p><img src="https://img-blog.csdn.net/20160619095143311" alt="TestJNI" title=""></p>

<blockquote>
  <p>本地方法跟普通的Java方法的区别在于方法声明多了native关键字。</p>
</blockquote>

<p><strong>JNI层实现Native方法</strong></p>

<p><img src="https://img-blog.csdn.net/20160619095309266" alt="实现Native方法" title=""></p>

<blockquote>
  <p>这里我们制造一个Native Crash，空指针异常。</p>
</blockquote>

<p><strong>通过Java调用Native方法</strong></p>

<p><img src="https://img-blog.csdn.net/20160619095629973" alt="调用Native方法" title=""></p>

<blockquote>
  <p>要调用Native方法需要先加载我们开发好的so库，通过System.loadLibrary(“so名字”);来调用，然后在通过java调用声明的native方法。</p>
</blockquote>

<h2 id="native-crash如何分析"><a name="t3"></a>Native Crash如何分析？</h2>

<p>既然要分析就必须找到可以分析的东西，我们在分析Java层Crash的时候是通过logcat日志找到对应的出错代码，然而Native层Crash也是可以logcat日志来进行分析的。</p>

<p>这里我们截取上面制造的crash在logcat显示的日志：</p>

<p><img src="https://img-blog.csdn.net/20160619100201177" alt="出错信息" title=""></p>

<p>这个是什么鬼，看不懂啊有木有。这个出错信息是我们调用native函数时打印出来的日志，只是简单的描述出错信号，出错地址还有进程号，看这个是完全摸不着调的。不过系统还是会提供相关有用的日志，我们在Android Studio查看logcat的时候需要做一下过滤。</p>

<p><img src="https://img-blog.csdn.net/20160619100848962" alt="添加DEBUG过滤项" title=""></p>

<p>在logcat添加完”DEBUG”的过滤项之后，我们就能得到以下log：</p>

<p><img src="https://img-blog.csdn.net/20160619101036260" alt="DEBUG" title=""></p>

<p>这下子可分析的内容就多起来了，我们逐个来看看： <br>
- 进程信息：pid表示进程号，tid表示线程号，name表示进程名 <br>
- 错误信号：signal 11表示信号的数字，SIGSEGV表示信号的名字，code 1（SEGV_MAPERR）表示出错代码，fault addr 00000000 表示出错的地址。 <br>
- 寄存器快照：进程收到错误信号时保存下来的寄存器快照，一共有15个寄存器。 <br>
- 堆栈信息：##00表示栈顶，##01调用#00，以此往下都是嵌套的调用关系，直至到栈顶。</p>

<blockquote>
  <p>这里参考了：<a href="http://bugly.qq.com/bbs/forum.php?mod=viewthread&amp;tid=27&amp;extra=page=4" rel="nofollow" target="_blank">http://bugly.qq.com/bbs/forum.php?mod=viewthread&amp;tid=27&amp;extra=page%3D4</a></p>
</blockquote>

<p>我们在栈顶就已经看到我们出错的地方了：</p>

<pre class="prettyprint" name="code"><code class="hljs lasso has-numbering"><span class="hljs-variable">#00</span>  pc <span class="hljs-number">00000730</span>  /<span class="hljs-built_in">data</span>/app<span class="hljs-attribute">-lib</span>/com<span class="hljs-built_in">.</span>devilwwj<span class="hljs-built_in">.</span>jnidemo<span class="hljs-subst">-</span><span class="hljs-number">1</span>/libJNIDemo<span class="hljs-built_in">.</span>so (Java_com_devilwwj_jnidemo_TestJNI_createANativeCrash)</code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre>

<p>pc 00000730 表示出错的地址，后面可以看到我加载了libJNIDemo.so库，接着是我们前面声明的Native方法，通过这种方法我们就能准确的找到出错的地方。</p>

<p>从上面的分析我们可以看到，so库崩溃时会产生信号异常，如果我们能够捕获到信号异常，相当于我们也能够顾捕获到Android Native崩溃了。</p>

<h1 id="总结"><a name="t4"></a>总结</h1>

<p>关于Native Crash的特点、产生原因、分析过程已经给大家做了简单的分析，这一块内容是初学者在分析错误的时候最头痛的地方，因为他不知道如何下手，也希望通过这篇文章能帮助到大家对Native Crash分析有个初步的认识，关于这一块还有很多东西可以讲，比如具体的signal有哪些，Linux下的信号机制是怎样的，怎样才能够捕获到信号等等，关于Native层的Crash捕获，我们有没有第三方的开发工具能帮助到我们，这里就要隆重推荐大家使用<a href="http://bugly.qq.com/" rel="nofollow" target="_blank">Bugly</a>，可以说是业内领先的崩溃捕获工具，不仅能够帮助我们获取到完整的错误堆栈，还能够将出错的上下文环境参数（比如系统版本、设备信息、内存信息等）详细的展现出来，大家不妨可以尝试下。最后，感谢大家的阅读。</p>            </div>


