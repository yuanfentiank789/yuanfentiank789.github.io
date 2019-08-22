---
layout: post
title:  LeakCanary 内存泄露监测原理研究
date:   2019-08-09 1:05:00
catalog:  true
tags:
    - LeakCanary 
    
         
       

---


<div data-note-content="" class="show-content">
          <div class="show-content-free">
            <p>"Read the fucking source code" -- linus一句名言体现出了阅读源码的重要性，学习别人得代码是提升自己的重要途径。最近用到了LeakCanary，顺便看一下其代码，学习一下。<br>
LeakCanary是安卓中用来检测内存泄露的小工具，它能帮助我们提早发现代码中隐藏的bug, 降低应用中内存泄露以及OOM产生的概率。</p>
<p>废话不多说，关于LeakCanary的使用方法，其实很简单，如果我们只想检测Activity的内存泄露，而且只想使用其默认的报告方式，我们只需要在Application中加一行代码，</p>
<pre class="hljs cpp"><code class="cpp">LeakCanary.install(<span class="hljs-keyword">this</span>);
</code></pre>
<p>那我们今天阅读源码的切入点，就从这个静态方法开始。</p>
<pre class="hljs cpp"><code class="cpp"> <span class="hljs-comment">/**
   * Creates a {@link RefWatcher} that works out of the box, and starts watching activity
   * references (on ICS+).
   */</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> RefWatcher <span class="hljs-title">install</span><span class="hljs-params">(Application application)</span> </span>{
    <span class="hljs-keyword">return</span> install(application, DisplayLeakService.class,
        AndroidExcludedRefs.createAppDefaults().build());
  }
</code></pre>
<p>这个函数内部直接调用了另外一个重载的函数</p>
<pre class="hljs php"><code class="php"><span class="hljs-comment">/**
   * Creates a {<span class="hljs-doctag">@link</span> RefWatcher} that reports results to the provided service, and starts watching
   * activity references (on ICS+).
   */</span>
  <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> RefWatcher install(Application application,
      <span class="hljs-class"><span class="hljs-keyword">Class</span>&lt;? <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractAnalysisResultService</span>&gt; <span class="hljs-title">listenerServiceClass</span>,
      <span class="hljs-title">ExcludedRefs</span> <span class="hljs-title">excludedRefs</span>) </span>{
    <span class="hljs-comment">//判断是否在Analyzer进程里</span>
    <span class="hljs-keyword">if</span> (isInAnalyzerProcess(application)) {
      <span class="hljs-keyword">return</span> RefWatcher.DISABLED;
    }
    enableDisplayLeakActivity(application);
    HeapDump.Listener heapDumpListener =
        <span class="hljs-keyword">new</span> ServiceHeapDumpListener(application, listenerServiceClass);
    RefWatcher refWatcher = androidWatcher(application, heapDumpListener, excludedRefs);
    ActivityRefWatcher.installOnIcsPlus(application, refWatcher);
    <span class="hljs-keyword">return</span> refWatcher;
  }
</code></pre>
<p>因为leakcanay会开启一个远程service用来分析每次产生的内存泄露，而安卓的应用每次开启进程都会调用Applicaiton的onCreate方法，因此我们有必要预先判断此次Application启动是不是在analyze service启动时，</p>
<pre class="hljs php"><code class="php"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> boolean isInServiceProcess(Context context, <span class="hljs-class"><span class="hljs-keyword">Class</span>&lt;? <span class="hljs-keyword">extends</span> <span class="hljs-title">Service</span>&gt; <span class="hljs-title">serviceClass</span>) </span>{
    PackageManager packageManager = context.getPackageManager();
    PackageInfo packageInfo;
    <span class="hljs-keyword">try</span> {
      packageInfo = packageManager.getPackageInfo(context.getPackageName(), GET_SERVICES);
    } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">Exception</span> e) {
      Log.e(<span class="hljs-string">"AndroidUtils"</span>, <span class="hljs-string">"Could not get package info for "</span> + context.getPackageName(), e);
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }
    String mainProcess = packageInfo.applicationInfo.processName;

    ComponentName component = <span class="hljs-keyword">new</span> ComponentName(context, serviceClass);
    ServiceInfo serviceInfo;
    <span class="hljs-keyword">try</span> {
      serviceInfo = packageManager.getServiceInfo(component, <span class="hljs-number">0</span>);
    } <span class="hljs-keyword">catch</span> (PackageManager.NameNotFoundException ignored) {
      <span class="hljs-comment">// Service is disabled.</span>
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }

    <span class="hljs-keyword">if</span> (serviceInfo.processName.equals(mainProcess)) {
      Log.e(<span class="hljs-string">"AndroidUtils"</span>,
          <span class="hljs-string">"Did not expect service "</span> + serviceClass + <span class="hljs-string">" to run in main process "</span> + mainProcess);
      <span class="hljs-comment">// Technically we are in the service process, but we're not in the service dedicated process.</span>
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }

    <span class="hljs-comment">//查找当前进程名</span>
    int myPid = android.os.Process.myPid();
    ActivityManager activityManager =
        (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
    ActivityManager.RunningAppProcessInfo myProcess = <span class="hljs-keyword">null</span>;
    <span class="hljs-keyword">for</span> (ActivityManager.RunningAppProcessInfo process : activityManager.getRunningAppProcesses()) {
      <span class="hljs-keyword">if</span> (process.pid == myPid) {
        myProcess = process;
        <span class="hljs-keyword">break</span>;
      }
    }
    <span class="hljs-keyword">if</span> (myProcess == <span class="hljs-keyword">null</span>) {
      Log.e(<span class="hljs-string">"AndroidUtils"</span>, <span class="hljs-string">"Could not find running process for "</span> + myPid);
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }

    <span class="hljs-keyword">return</span> myProcess.processName.equals(serviceInfo.processName);
  }
</code></pre>
<p>判断Application是否是在service进程里面启动，最直接的方法就是判断当前进程名和service所属的进程是否相同。当前进程名的获取方式是使用ActivityManager的getRunningAppProcessInfo方法，找到进程pid与当前进程pid相同的进程，然后从中拿到processName. service所属进程名。获取service应处进程的方法是用PackageManager的getPackageInfo方法。</p>
<h3>RefWatcher</h3>
<p>ReftWatcher是leakcancay检测内存泄露的发起点。使用方法为，在对象生命周期即将结束的时候，调用</p>
<pre class="hljs javascript"><code class="javascript">RefWatcher.watch(<span class="hljs-built_in">Object</span> object)
</code></pre>
<p>为了达到检测内存泄露的目的，RefWatcher需要</p>
<pre class="hljs java"><code class="java">  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Executor watchExecutor;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> DebuggerControl debuggerControl;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> GcTrigger gcTrigger;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> HeapDumper heapDumper;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Set&lt;String&gt; retainedKeys;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ReferenceQueue&lt;Object&gt; queue;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> HeapDump.Listener heapdumpListener;
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ExcludedRefs excludedRefs;
</code></pre>
<ul>
<li>watchExecutor: 执行内存泄露检测的executor</li>
<li>debuggerControl ：用于查询是否正在调试中，调试中不会执行内存泄露检测</li>
<li>queue ： 用于判断弱引用所持有的对象是否已被GC。</li>
<li>gcTrigger： 用于在判断内存泄露之前，再给一次GC的机会</li>
<li>headDumper: 用于在产生内存泄露室执行dump 内存heap</li>
<li>heapdumpListener: 用于分析前面产生的dump文件，找到内存泄露的原因</li>
<li>excludedRefs: 用于排除某些系统bug导致的内存泄露</li>
<li>retainedKeys： 持有那些呆检测以及产生内存泄露的引用的key。</li>
</ul>
<p>接下来，我们来看看watch函数背后是如何利用这些工具，生成内存泄露分析报告的。</p>
<pre class="hljs java"><code class="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">watch</span><span class="hljs-params">(Object watchedReference, String referenceName)</span> </span>{
    checkNotNull(watchedReference, <span class="hljs-string">"watchedReference"</span>);
    checkNotNull(referenceName, <span class="hljs-string">"referenceName"</span>);
    <span class="hljs-comment">//如果处于debug模式，则直接返回</span>
    <span class="hljs-keyword">if</span> (debuggerControl.isDebuggerAttached()) {
      <span class="hljs-keyword">return</span>;
    }
    <span class="hljs-comment">//记住开始观测的时间</span>
    <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> watchStartNanoTime = System.nanoTime();
    <span class="hljs-comment">//生成一个随机的key，并加入set中</span>
    String key = UUID.randomUUID().toString();
    retainedKeys.add(key);
    <span class="hljs-comment">//生成一个KeyedWeakReference</span>
    <span class="hljs-keyword">final</span> KeyedWeakReference reference =
        <span class="hljs-keyword">new</span> KeyedWeakReference(watchedReference, key, referenceName, queue);
    <span class="hljs-comment">//调用watchExecutor，执行内存泄露的检测</span>
    watchExecutor.execute(<span class="hljs-keyword">new</span> Runnable() {
      <span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
        ensureGone(reference, watchStartNanoTime);
      }
    });
  }
</code></pre>
<p>所以最后的核心函数是在ensureGone这个runnable里面。要理解其工作原理，就得从keyedWeakReference说起</p>
<h3>WeakReference与ReferenceQueue</h3>
<p>从watch函数中，可以看到，每次检测对象内存是否泄露时，我们都会生成一个KeyedReferenceQueue，这个类其实就是一个WeakReference，只不过其额外附带了一个key和一个name</p>
<pre class="hljs javascript"><code class="javascript">final <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KeyedWeakReference</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WeakReference</span>&lt;<span class="hljs-title">Object</span>&gt; </span>{
  public final <span class="hljs-built_in">String</span> key;
  public final <span class="hljs-built_in">String</span> name;

  KeyedWeakReference(<span class="hljs-built_in">Object</span> referent, <span class="hljs-built_in">String</span> key, <span class="hljs-built_in">String</span> name,
      ReferenceQueue&lt;<span class="hljs-built_in">Object</span>&gt; referenceQueue) {
    <span class="hljs-keyword">super</span>(checkNotNull(referent, <span class="hljs-string">"referent"</span>), checkNotNull(referenceQueue, <span class="hljs-string">"referenceQueue"</span>));
    <span class="hljs-keyword">this</span>.key = checkNotNull(key, <span class="hljs-string">"key"</span>);
    <span class="hljs-keyword">this</span>.name = checkNotNull(name, <span class="hljs-string">"name"</span>);
  }
}
</code></pre>
<p>在构造时我们需要传入一个ReferenceQueue，这个ReferenceQueue是直接传入了WeakReference中，关于这个类，有兴趣的可以直接看Reference的源码。我们这里需要知道的是，每次WeakReference所指向的对象被GC后，这个弱引用都会被放入这个与之相关联的ReferenceQueue队列中。</p>
<p>我们这里可以贴下其核心代码</p>
<pre class="hljs java"><code class="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReferenceHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Thread</span> </span>{

        ReferenceHandler(ThreadGroup g, String name) {
            <span class="hljs-keyword">super</span>(g, name);
        }

        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
            <span class="hljs-keyword">for</span> (;;) {
                Reference&lt;Object&gt; r;
                <span class="hljs-keyword">synchronized</span> (lock) {
                    <span class="hljs-keyword">if</span> (pending != <span class="hljs-keyword">null</span>) {
                        r = pending;
                        pending = r.discovered;
                        r.discovered = <span class="hljs-keyword">null</span>;
                    } <span class="hljs-keyword">else</span> {
                        <span class="hljs-comment">//....</span>
                        <span class="hljs-keyword">try</span> {
                            <span class="hljs-keyword">try</span> {
                                lock.wait();
                            } <span class="hljs-keyword">catch</span> (OutOfMemoryError x) { }
                        } <span class="hljs-keyword">catch</span> (InterruptedException x) { }
                        <span class="hljs-keyword">continue</span>;
                    }
                }

                <span class="hljs-comment">// Fast path for cleaners</span>
                <span class="hljs-keyword">if</span> (r <span class="hljs-keyword">instanceof</span> Cleaner) {
                    ((Cleaner)r).clean();
                    <span class="hljs-keyword">continue</span>;
                }

                ReferenceQueue&lt;Object&gt; q = r.queue;
                <span class="hljs-keyword">if</span> (q != ReferenceQueue.NULL) q.enqueue(r);
            }
        }
    }

    <span class="hljs-keyword">static</span> {
        ThreadGroup tg = Thread.currentThread().getThreadGroup();
        <span class="hljs-keyword">for</span> (ThreadGroup tgn = tg;
             tgn != <span class="hljs-keyword">null</span>;
             tg = tgn, tgn = tg.getParent());
        Thread handler = <span class="hljs-keyword">new</span> ReferenceHandler(tg, <span class="hljs-string">"Reference Handler"</span>);
        <span class="hljs-comment">/* If there were a special system-only priority greater than
         * MAX_PRIORITY, it would be used here
         */</span>
        handler.setPriority(Thread.MAX_PRIORITY);
        handler.setDaemon(<span class="hljs-keyword">true</span>);
        handler.start();
    }
</code></pre>
<p>在reference类加载的时候，java虚拟机会创建一个最大优先级的后台线程，这个线程的工作原理就是不断检测pending是否为null，如果不为null，就将其放入ReferenceQueue中，pending不为null的情况就是，引用所指向的对象已被GC，变为不可达。</p>
<p>那么只要我们在构造弱引用的时候指定了ReferenceQueue，每当弱引用所指向的对象被内存回收的时候，我们就可以在queue中找到这个引用。如果我们期望一个对象被回收，那如果在接下来的预期时间之后，我们发现它依然没有出现在ReferenceQueue中，那就可以判定它的内存泄露了。LeakCanary检测内存泄露的核心原理就在这里。</p>
<p>其实Java里面的WeakHashMap里也用到了这种方法，来判断hash表里的某个键值是否还有效。在构造WeakReference的时候给其指定了ReferenceQueue.</p>
<h3>监测时机</h3>
<p>什么时候去检测能判定内存泄露呢？这个可以看AndroidWatchExecutor的实现</p>
<pre class="hljs java"><code class="java">
<span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AndroidWatchExecutor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Executor</span> </span>{

    <span class="hljs-comment">//....</span>
    
    <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">executeDelayedAfterIdleUnsafe</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Runnable runnable)</span> </span>{
        <span class="hljs-comment">// This needs to be called from the main thread.</span>
        Looper.myQueue().addIdleHandler(<span class="hljs-keyword">new</span> MessageQueue.IdleHandler() {
          <span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">queueIdle</span><span class="hljs-params">()</span> </span>{
            backgroundHandler.postDelayed(runnable, DELAY_MILLIS);
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
          }
        });
      }
  }
</code></pre>
<p>这里又看到一个比较少的用法，<strong>IdleHandler</strong>，IdleHandler的原理就是在messageQueue因为空闲等待消息时给使用者一个hook。那AndroidWatchExecutor会在主线程空闲的时候，派发一个后台任务，这个后台任务会在DELAY_MILLIS时间之后执行。LeakCanary设置的是5秒。</p>
<h3>二次确认保证内存泄露准确性</h3>
<p>为了避免因为gc不及时带来的误判，leakcanay会进行二次确认进行保证。</p>
<pre class="hljs java"><code class="java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">ensureGone</span><span class="hljs-params">(KeyedWeakReference reference, <span class="hljs-keyword">long</span> watchStartNanoTime)</span> </span>{
    <span class="hljs-keyword">long</span> gcStartNanoTime = System.nanoTime();
    <span class="hljs-comment">//计算从调用watch到进行检测的时间段</span>
    <span class="hljs-keyword">long</span> watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);
    <span class="hljs-comment">//根据queue移除已被GC的对象的弱引用</span>
    removeWeaklyReachableReferences();
    <span class="hljs-comment">//如果内存已被回收或者处于debug模式，直接返回</span>
    <span class="hljs-keyword">if</span> (gone(reference) || debuggerControl.isDebuggerAttached()) {
      <span class="hljs-keyword">return</span>;
    }
    <span class="hljs-comment">//如果内存依旧没被释放，则再给一次gc的机会</span>
    gcTrigger.runGc();
    <span class="hljs-comment">//再次移除</span>
    removeWeaklyReachableReferences();
    <span class="hljs-keyword">if</span> (!gone(reference)) {
      <span class="hljs-comment">//走到这里，认为内存确实泄露了</span>
      <span class="hljs-keyword">long</span> startDumpHeap = System.nanoTime();
      <span class="hljs-keyword">long</span> gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);
      <span class="hljs-comment">//dump出heap报告</span>
      File heapDumpFile = heapDumper.dumpHeap();

      <span class="hljs-keyword">if</span> (heapDumpFile == HeapDumper.NO_DUMP) {
        <span class="hljs-comment">// Could not dump the heap, abort.</span>
        <span class="hljs-keyword">return</span>;
      }
      <span class="hljs-keyword">long</span> heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
      heapdumpListener.analyze(
          <span class="hljs-keyword">new</span> HeapDump(heapDumpFile, reference.key, reference.name, excludedRefs, watchDurationMs,
              gcDurationMs, heapDumpDurationMs));
    }
  }

  <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">gone</span><span class="hljs-params">(KeyedWeakReference reference)</span> </span>{
    <span class="hljs-keyword">return</span> !retainedKeys.contains(reference.key);
  }

  <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeWeaklyReachableReferences</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">// WeakReferences are enqueued as soon as the object to which they point to becomes weakly</span>
    <span class="hljs-comment">// reachable. This is before finalization or garbage collection has actually happened.</span>
    KeyedWeakReference ref;
    <span class="hljs-keyword">while</span> ((ref = (KeyedWeakReference) queue.poll()) != <span class="hljs-keyword">null</span>) {
      retainedKeys.remove(ref.key);
    }
  }
</code></pre>
<h3>Dump Heap</h3>
<p>监测到内存泄露后，首先做的就是dump出当前的heap，默认的AndroidHeapDumper调用的是</p>
<pre class="hljs css"><code class="css"><span class="hljs-selector-tag">Debug</span><span class="hljs-selector-class">.dumpHprofData</span>(<span class="hljs-selector-tag">filePath</span>);
</code></pre>
<p>到处当前内存的hprof分析文件，一般我们在DeviceMonitor中也可以dump出hprof文件，然后将其从dalvik格式转成标准jvm格式，然后使用MAT进行分析。</p>
<p>那么LeakCanary是如何分析内存泄露的呢？</p>
<h3>HaHa</h3>
<p>LeakCanary 分析内存泄露用到了一个和Mat类似的工具叫做<a href="https://link.jianshu.com?t=https://github.com/square/haha" target="_blank" rel="nofollow">HaHa</a>，使用HaHa的方法如下：</p>
<pre class="hljs php"><code class="php"><span class="hljs-keyword">public</span> AnalysisResult checkForLeak(File heapDumpFile, String referenceKey) {
    long analysisStartNanoTime = System.nanoTime();

    <span class="hljs-keyword">if</span> (!heapDumpFile.exists()) {
      <span class="hljs-keyword">Exception</span> <span class="hljs-keyword">exception</span> = <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"File does not exist: "</span> + heapDumpFile);
      <span class="hljs-keyword">return</span> failure(<span class="hljs-keyword">exception</span>, since(analysisStartNanoTime));
    }

    <span class="hljs-keyword">try</span> {
      HprofBuffer buffer = <span class="hljs-keyword">new</span> MemoryMappedFileBuffer(heapDumpFile);
      HprofParser parser = <span class="hljs-keyword">new</span> HprofParser(buffer);
      Snapshot snapshot = parser.parse();

      Instance leakingRef = findLeakingReference(referenceKey, snapshot);

      <span class="hljs-comment">// False alarm, weak reference was cleared in between key check and heap dump.</span>
      <span class="hljs-keyword">if</span> (leakingRef == <span class="hljs-keyword">null</span>) {
        <span class="hljs-keyword">return</span> noLeak(since(analysisStartNanoTime));
      }

      <span class="hljs-keyword">return</span> findLeakTrace(analysisStartNanoTime, snapshot, leakingRef);
    } <span class="hljs-keyword">catch</span> (Throwable e) {
      <span class="hljs-keyword">return</span> failure(e, since(analysisStartNanoTime));
    }
  }
</code></pre>
<p>关于HaHa的原理，感兴趣的同学可以深究，这里就不深入介绍了。</p>
<p>返回的ActivityResult对象中包含了对象到GC root的最短路径。LeakCanary在dump出hprof文件后，会启动一个IntentService进行分析：HeapAnalyzerService在分析出结果之后会启动DisplayLeakService用来发起Notification 以及将结果记录下来写在文件里面。以后每次启动LeakAnalyzerActivity就从文件里读取历史结果。</p>
<h3>ExcludedRef</h3>
<p>由于某些系统的bug，以及某些厂商rom的bug，Activity在finish之后仍然会被某些系统组件给hold住。LeakCanary列出了一些很常见的，比如三星的手机activity会被audioManager给hold住，试了一下huawei的系统貌似也会出现，还有比如activity中如果有会获取键盘焦点的view，在activity finish之后view会被InputMethodManager给hold住，因为view会持有activity 造成activity泄漏，除非有新的view获取键盘焦点。</p>
<p>LeakCanary中有一个AndroidExcludedRefs枚举类，其中枚举了很多特定版本系统issue引起的内存泄漏，因为这种问题 不是开发者导致的，因此HeapAnalyzerService在分析内存泄露时，会将这些GC Root排除在外。而且每个ExcludedRef通常都跟特定厂商或者Android版本有关，这些枚举类都加了一个适用条件。</p>
<pre class="hljs java"><code class="java">AndroidExcludedRefs(<span class="hljs-keyword">boolean</span> applies) {  <span class="hljs-keyword">this</span>.applies = applies;}


   AUDIO_MANAGER__MCONTEXT_STATIC(SAMSUNG.equals(MANUFACTURER) &amp;&amp; SDK_INT == KITKAT) {
    <span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">add</span><span class="hljs-params">(ExcludedRefs.Builder excluded)</span> </span>{
      <span class="hljs-comment">// Samsung added a static mContext_static field to AudioManager, holds a reference to the</span>
      <span class="hljs-comment">// activity.</span>
      <span class="hljs-comment">// Observed here: https://github.com/square/leakcanary/issues/32</span>
      excluded.staticField(<span class="hljs-string">"android.media.AudioManager"</span>, <span class="hljs-string">"mContext_static"</span>);
    }
  },
</code></pre>
<p>比如上面这个AudioManager引起的问题，只有在Build中的MANUFACTURER表明是三星以及sdk版本是KITKAT（4.4, 19)时才适用。</p>
<h3>手动释放资源</h3>
<p>然后并不是leakCanary不报错我们就不用管，activity内存泄露了，大部分情况下没多大事，但是有些占用内存很多的页面，比如图库，webview页面，因为acitivity不能回收，它所指向的view以及view下面的bitmap都不能被回收，这是会造成很不好的后果的，很可能会导致OOM，因此我们需要手动在Activity结束时回收资源。</p>
<h3>Under 4.0 &amp; Fragment</h3>
<p>LeakCanary只支持4.0以上，原因是其中在watch 每个Activity时适用了Application的<strong>registerActivityLifecycleCallback</strong>函数，这个函数只在4.0上才支持，但是在4.0以下也是可以用的，可以在Application中将返回的RefWatcher存下来，然后在基类Activity的onDestroy函数中调用。</p>
<p>同理，如果我们想检测Fragment的内存的话，我们也阔以在Fragment的onDestroy中watch它。</p>

          </div>
        

