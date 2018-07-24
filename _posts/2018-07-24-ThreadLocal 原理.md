---
layout: post
title:  "ThreadLocal 原理"
date:   2018-07-24 1:05:00
catalog:  true
tags:
    - ThreadLocal


---

<div data-v-41d33d72="" itemprop="articleBody" class="entry-content article-content"><p>ThreadLocal 可以把一个对象保存在指定的线程中，对象保存后，只能在指定线程中获取保存的数据，对于其他线程来说则无法获取到数据。日常开发中 ThreadLocal 使用的地方比较少，但是系统在 Handler 机制中使用了它来保证每一个 Handler 所在的线程中都有一个独立的 Looper 对象，为了更好的理解 Handler 机制，这篇文章来说说 ThreadLocal 的原理，一窥究竟。</p>
<h2 id="directory082497352325284421" data-id="heading-0">ThreadLocal 是什么</h2>
<p>ThreadLocal 位于 java.lang 包下。</p>
<p>ThreadLocal 是一个关于创建线程局部变量的类。什么是线程的局部变量呢？其实就是这个变量的作用域是线程，其他线程访问不了。通常我们创建的变量是可以被任何一个线程访问的，而使用 ThreadLocal 创建的变量只能被当前线程访问，其他线程无法访问。</p>
<h2 id="directory082497352325284422" data-id="heading-1">使用示例</h2>
<p>先来看看一个使用 ThreadLocal 的示例，对 ThreadLocal 有一个基本、直观的认识。</p><pre><code>public class ThreadLocalTest extends AppCompatActivity {
    private static final String TAG = "ThreadLocalTest";
    private ThreadLocal&lt;String&gt; stringThreadLocal = new ThreadLocal&lt;&gt;();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        booleanThreadLocal.set("MainThread");
        Log.d(TAG, "MainThread's stringThreadLocal=" + stringThreadLocal.get());
        new Thread("Thread#1") {
            @Override
            public void run() {
                Log.d(TAG, "Thread#1's stringThreadLocal=" + stringThreadLocal.get());
            }
        }.start();
    }
}</code></pre>
<p>首先创建了一个 泛型为 String 的 ThreadLocal 对象，并初始化。这样就有了一个可以保存 String 类型的 ThreadLocal 对象。接着在主线程和子线程中分别操作该对象，使用 set 方法赋值，get 方法取值，注意看每个线程中的打印结果。</p><pre><code>D/ThreadLocalTest: MainThread's stringThreadLocal = MainThread
D/ThreadLocalTest: Thread#1's stringThreadLocal = null</code></pre>
<p>可以看到，MainThread 对 stringThreadLocal 的修改并没有影响到 Thread#1 中的值。说明了使用 ThreadLocal 保存的对象的作用域是当前线程。</p>
<h2 id="directory082497352325284423" data-id="heading-2">Looper 中的使用</h2>
<p>再来看看 Android 源码中 Looper.java 是怎样使用 ThreadLocal 的。</p><pre><code>// sThreadLocal.get() will return null unless you've called prepare().
static final ThreadLocal&lt;Looper&gt; sThreadLocal = new ThreadLocal&lt;Looper&gt;();
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}</code></pre>
<p>这里使用 ThreadLocal 保存 Looper，确保每个线程中只有一个 Looper 对象。</p>
<h2 id="directory082497352325284424" data-id="heading-3">修改默认值</h2>
<p>从上面的示例中，我们看到 ThreadLocal 保存的对象默认值是 null。如果我们需要给定一个默认的值，就需要重写 initialValue 方法，该方法默认返回 null，我们可以根据具体要求返回需要的值，如下所示。</p><pre><code>private ThreadLocal&lt;Boolean&gt; booleanThreadLocal = new ThreadLocal&lt;Boolean&gt;(){
    @Override
    protected Boolean initialValue() {
        return false;
    }
};</code></pre>
<p>ThreadLocal 还有一个对外提供的方法 remove，看名字就知道这是删除已经保存的数据的。</p>
<h2 id="directory082497352325284425" data-id="heading-4">原理</h2>
<h3 id="directory082497352325284426" data-id="heading-5">set 方法</h3>
<p>ThreadLocal 的 public 方法，只有三个：set、get、remove。我们先从 set 方法入手。</p><pre><code>public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}</code></pre>
<p>set 方法进行了如下几部操作：</p>
<p>1.获取当前线程<br>2.使用当前线程获取一个 ThreadLocalMap 对象<br>3.如果获取到的 map 对象不为空，则设置值，否则创建 map 设置值</p>
<p>下面是 getMap 源码：</p><pre><code>ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}</code></pre>
<p>上面代码获取到的是 Thread 对象的 threadLocals 变量，类型为 ThreadLocal.ThreadLocalMap。</p>
<p>而如果 map 对象为空，则新建 ThreadLocalMap 对象。</p><pre><code>void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}</code></pre>
<p>结论：原来每个线程都有一个保存值的 ThreadLocalMap 对象，ThreadLocal 的值就存放在了当前线程的 ThreadLocalMap 成员变量中，所以只能在本线程访问，其他线程不能访问。</p>
<p>我们在看看具体的保存方法：ThreadLocalMap#set</p><pre><code>private void set(ThreadLocal key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode &amp; (len-1);

    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
        ThreadLocal k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) &amp;&amp; sz &gt;= threshold)
        rehash();
}</code></pre>
<p>上面的代码实现了数据的存储，其中 table 是一个 Entry[] 数组对象，而 Entry 是用来存储 ThreadLocal key, Object value 的，逻辑是根据 key 找出 Entry 对象，如果找出的这个 Entry 的 k 等于 key，直接设置 Entry 的 value，如果 k 为空，则通过 replaceStaleEntry 保存数据，最后构建出 Entry 保存进 table 数组中。</p>
<p>Entry 对象是怎样保存 key 和 value 的呢？</p><pre><code>static class Entry extends WeakReference&lt;ThreadLocal&gt; {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}</code></pre>
<p>原来 Entry 继承了 WeakReference&lt;ThreadLocal&gt;，那么通过 Entry 对象的 get 方法就可以获取到一个弱引用的 ThreadLocal 对象。扩展了一个 Object 类型的 value 对象，并且在构造方法中进行了初始化赋值。Entry 保存了 ThreadLocal(key) 和 对应的值(value)，其中 ThreadLoacl 是通过弱引用的形式，避免了线程池线程复用带来的内存泄露。</p>
<h3 id="directory082497352325284427" data-id="heading-6">get 方法</h3>
<p>看完 set 方法，再来看看 get 方法的源码：</p><pre><code>public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}</code></pre>
<p>get 方法首先取出当前线程的 ThreadLocalMap 对象，如果这个对象为空，则返回默认值；如果不为空，使用当前 ThreadLoacl 对象(this)获取 ThreadLocalMap 的 Entry 对象，返回 Entry 保存的 value 值。 </p>
<p>从 ThreadLoacl 的 set 和 get 方法来看，它们操作的对象都是当前线程对象中的 ThreadLocalMap 对象的 Entry[] 数组，因此在不同的线程中访问同一个 ThreadLoacl 的 set 和 get 方法，操作的对应线程中的数据，所以不会影响到其他线程。</p>
<p>参考：</p>
<p><a href="https://link.juejin.im?target=http%3A%2F%2Fdroidyue.com%2Fblog%2F2016%2F03%2F13%2Flearning-threadlocal-in-java%2Findex.html" target="_blank" rel="nofollow noopener noreferrer">理解Java中的ThreadLocal</a><br><a href="https://link.juejin.im?target=https%3A%2F%2Fbook.douban.com%2Fsubject%2F26599538%2F" target="_blank" rel="nofollow noopener noreferrer">Android开发艺术探索-10.2.1章节</a></p></div>

## 测试代码
```

class ThreadLocalDemo{
    public static void main (String[] args){
       ThreadLocal<String> local1 = new ThreadLocal<String>(){
           @Override
           protected String initialValue() {
               return "default value"; //String类型的默认值
           }
       };
       ThreadLocal<Boolean> local2 = new ThreadLocal<Boolean>(){
           @Override
           protected Boolean initialValue() {
               return true;
           }
       };
       //local1.set("hh1");//通过threadlocal在一个线程每种类型只能存储一个值
        // 每个线程可以有多个threadlocal对象，因为所有threadlocal最终操作的
        // 都是thread的threadLocals成员变量，类型为threadlocalMap
       System.out.println(local1.get());
       System.out.println(local2.get());
    }
}
```

