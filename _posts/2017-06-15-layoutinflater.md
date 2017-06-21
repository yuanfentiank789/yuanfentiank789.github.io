---

layout: post
title:  "LayoutInflater源码分析与应用"
date:   2017-06-15 1:05:00
catalog:  true
tags:

   - LayoutInflater
   
   
       
   
---

<div class="article">
        <h1 class="title">LayoutInflater源码分析与应用</h1>

        <!-- 作者区域 -->
        <div class="author">
          <a class="avatar" href="/u/f9de259236a3">
            <img src="//upload.jianshu.io/users/upload_avatars/4050443/b7e70f29-5836-4b77-8c49-40321db03541.png?imageMogr2/auto-orient/strip|imageView2/1/w/144/h/144" alt="144">
</a>          <div class="info">
            <span class="tag">作者</span>
            <span class="name"><a href="/u/f9de259236a3">CSDN_LQR</a></span>
            <!-- 关注用户按钮 -->
            <a class="btn btn-success follow"><i class="iconfont ic-follow"></i><span>关注</span></a>
            <!-- 文章数据信息 -->
            <div class="meta">
              <!-- 如果文章更新时间大于发布时间，那么使用 tooltip 显示更新时间 -->
                <span class="publish-time" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="最后编辑于 2017.05.04 08:31">2017.04.23 13:21*</span>
              <span class="wordage">字数 1845</span>
            <span class="views-count">阅读 426</span><span class="comments-count">评论 6</span><span class="likes-count">喜欢 6</span></div>
          </div>
          <!-- 如果是当前作者，加入编辑按钮 -->
        </div>
        <!-- -->

        <!-- 文章内容 -->
        <div class="show-content">
          <blockquote><p>转载自[http://www.jianshu.com/p/de7f651170be] </p></blockquote>
<h1>一、简述</h1>
<p>LayoutInflater直译为 布局填充器，它是用来创建布局视图的，常用inflate()将一个xml布局文件转换成一个View，下面先介绍下获取LayoutInflater的三种方式 和 创建View的两种方式。</p>
<h2>1、获取LayoutInflater的三种方式</h2>
<ol>
<li>LayoutInflater inflater = getLayoutInflater();  //调用Activity的getLayoutInflater()</li>
<li>LayoutInflater inflater =(LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);</li>
<li>LayoutInflater inflater = LayoutInflater.from(context);  </li>
</ol>
<p>其实不管是哪种方式，最后都是通过方式2获取到LayoutInflater的，如：</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-37ec4a4200899cef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-37ec4a4200899cef.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<h2>2、创建View的两种方式</h2>
<ol>
<li>View.inflate();</li>
<li>LayoutInflater.from(context).inflate();</li>
</ol>
<h1>二、源码分析</h1>
<p>上面两种创建View的方式都是开发中常用的，那两者有什么关系吗？下面对View.inflate()进行方法调用分析：</p>
<h2>1、View.inflate()最终调用方法探究</h2>
<h3>1）按住Ctrl+鼠标左键查看View.inflate()方法</h3>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-444635360bcd95da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-444635360bcd95da.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>可以看到View.inflate()就是调用了LayoutInflater.from(context).inflate()。</p>
<p>好，到这一步要明确，不管我们研究哪种方式，实际上都研究方式2，即LayoutInflater.from(context).inflate()。</p>
<h3>2）按住Ctrl+鼠标左键查看LayoutInflater.from(context).inflate(resource, root)方法。</h3>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-3ee8975fdfa143dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-3ee8975fdfa143dd.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>嗯？LayoutInflater.from(context).inflate(resource, root)再调用了自己的重载inflate(resource, root, root != null)。</p>
<h3>3）按住Ctrl+鼠标左键查看LayoutInflater.from(context).inflate(resource, root).inflate(resource, root, root != null)方法。</h3>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-c07c5a5e6d9b12fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-c07c5a5e6d9b12fa.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>嗯？？LayoutInflater.from(context).inflate(resource, root).inflate(resource, root, root != null)再再调用了自己的重载inflate(parser, root, attachToRoot)。</p>
<h3>4）按住Ctrl+鼠标左键查看LayoutInflater.from(context).inflate(resource, root).inflate(resource, root, root != null).inflate(parser, root, attachToRoot)方法。</h3>
<p>这下总算是到头了，不过代码太长，这里就截了一半的图（这不是重点）。</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-260aba43dd5938fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-260aba43dd5938fe.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>好，重点来了，到这步我们可以明白一点，View.inflate()整个方法调用链如下：</p>
<pre class="hljs x86asm"><code class="x86asm">View.inflate() = 
    LayoutInflater.from(context)
<span class="hljs-meta">        .inflate</span>(resource, root)
<span class="hljs-meta">        .inflate</span>(resource, root, root != null)
<span class="hljs-meta">        .inflate</span>(parser, root, attachToRoot)</code></pre>
<h2>2、LayoutInflater的inflate(parser, root, attachToRoot)做了什么？</h2>
<p>由于代码太长，不方便截图，下面贴出代码中的重点代码：</p>
<pre class="hljs lasso"><code class="lasso"><span class="hljs-keyword">public</span> View inflate(XmlPullParser parser, @Nullable ViewGroup root, <span class="hljs-built_in">boolean</span> attachToRoot) {
    synchronized (mConstructorArgs) {

                <span class="hljs-params">...</span>
                省略代码~
                <span class="hljs-params">...</span>

                final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                ViewGroup.LayoutParams <span class="hljs-keyword">params</span> = <span class="hljs-built_in">null</span>;

                <span class="hljs-keyword">if</span> (root != <span class="hljs-built_in">null</span>) {
                    <span class="hljs-keyword">if</span> (DEBUG) {
                        System.out.println(<span class="hljs-string">"Creating params from root: "</span> +
                                root);
                    }
                    <span class="hljs-comment">// Create layout params that match root, if supplied</span>
                    <span class="hljs-keyword">params</span> = root.generateLayoutParams(attrs);
                    <span class="hljs-keyword">if</span> (!attachToRoot) {
                        <span class="hljs-comment">// Set the layout params for temp if we are not</span>
                        <span class="hljs-comment">// attaching. (If we are, we use addView, below)</span>
                        temp.setLayoutParams(<span class="hljs-keyword">params</span>);
                    }
                }

                <span class="hljs-params">...</span>
                省略代码~
                <span class="hljs-params">...</span>

                <span class="hljs-comment">// We are supposed to attach all the views we found (int temp)</span>
                <span class="hljs-comment">// to root. Do that now.</span>
                <span class="hljs-keyword">if</span> (root != <span class="hljs-built_in">null</span> &amp;&amp; attachToRoot) {
                    root.addView(temp, <span class="hljs-keyword">params</span>);
                }

                <span class="hljs-comment">// Decide whether to return the root that was passed in or the</span>
                <span class="hljs-comment">// top view found in xml.</span>
                <span class="hljs-keyword">if</span> (root == <span class="hljs-built_in">null</span> || !attachToRoot) {
                    result = temp;
                }

                <span class="hljs-params">...</span>
                省略代码~
                <span class="hljs-params">...</span>

        <span class="hljs-keyword">return</span> result;
    }
}</code></pre>
<p>该inflate方法中有以下四步操作：</p>
<ol>
<li>通过使用XmlPullParser parser将xml布局文件转换成视图temp。</li>
<li>判断ViewGroup root对象是否为null，来决定要不要给temp设置LayoutParams。</li>
<li>判断boolean attachToRoot是否为true，来决定是否要把temp顺便加到ViewGroup root中。</li>
<li>最后返回视图temp。</li>
</ol>
<p>到这里就把创建视图的流程分析完了，接下来是比较 View.inflate()和 LayoutInflater.from(context).inflate()的区别。</p>
<h2>3、View.inflate()和 LayoutInflater.from(context).inflate()的区别</h2>
<h3>1）View.inflate()第三个参数的解析：</h3>
<p>开发中常常会对第三个参数（ViewGroup root）传入null吧，通过上面对最终inflate方法的分析，可以知道，如果ViewGroup root取值为null，则得到的视图temp不会被设置LayoutParams。下面做个试验：</p>
<pre class="hljs lasso"><code class="lasso">View itemView = View.inflate(<span class="hljs-keyword">parent</span>.getContext(), android.R.layout.simple_list_item_1, <span class="hljs-built_in">null</span>);
ViewGroup.LayoutParams <span class="hljs-keyword">params</span> = itemView.getLayoutParams();
<span class="hljs-keyword">Log</span>.e(<span class="hljs-string">"CSDN_LQR"</span>, <span class="hljs-string">"params == null : "</span> + (<span class="hljs-keyword">params</span> == <span class="hljs-built_in">null</span>));</code></pre>
<p>打印结果如下：</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-337f2ed3b39b9345.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-337f2ed3b39b9345.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>同理，将第三个参数传入一个确实存在的ViewGroup时，结果就是视图temp能获取到LayoutParams，有兴趣的可以自己试试。</p>
<h3>2）LayoutInflater.from(context).inflate()的优势：</h3>
<p><em>*下面的场景分析将体现出LayoutInflater.from(context).inflate()的灵活性。</em></p>
<p>如果是在RecyclerView或ListView中使用View.inflate()创建布局视图，又想对创建出来的布局视图进行高度等参数设置时，会有什么瓶颈呢？</p>
<p>下面贴出我之前写过的一段用于瀑布流适配器的代码：</p>
<pre class="hljs scala"><code class="scala">public <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyStaggeredAdapter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RecyclerView</span>.<span class="hljs-title">Adapter&lt;MyStaggeredAdapter</span>.<span class="hljs-title">MyViewHolder&gt;</span> </span>{

    <span class="hljs-keyword">private</span> <span class="hljs-type">List</span>&lt;<span class="hljs-type">String</span>&gt; mData;
    <span class="hljs-keyword">private</span> <span class="hljs-type">Random</span> mRandom = <span class="hljs-keyword">new</span> <span class="hljs-type">Random</span>();

    public <span class="hljs-type">MyStaggeredAdapter</span>(<span class="hljs-type">List</span>&lt;<span class="hljs-type">String</span>&gt; data) {
        mData = data;
    }

    <span class="hljs-meta">@Override</span>
    public <span class="hljs-type">MyViewHolder</span> onCreateViewHolder(<span class="hljs-type">ViewGroup</span> parent, int viewType) {
        <span class="hljs-comment">//这里使用的是安卓自带的文本控件布局</span>
        <span class="hljs-type">View</span> itemView = <span class="hljs-type">View</span>.inflate(parent.getContext(), android.<span class="hljs-type">R</span>.layout.simple_list_item_1, <span class="hljs-literal">null</span>);
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-type">MyViewHolder</span>(itemView);
    }

    <span class="hljs-meta">@Override</span>
    public void onBindViewHolder(<span class="hljs-type">MyViewHolder</span> holder, int position) {
        <span class="hljs-comment">//为实现瀑布流效果，需要对条目高度进行设置（让各个条目的高度不同）</span>
        <span class="hljs-type">ViewGroup</span>.<span class="hljs-type">LayoutParams</span> params = holder.mTv.getLayoutParams();
        params.height = mRandom.nextInt(<span class="hljs-number">200</span>) + <span class="hljs-number">200</span>;
        holder.mTv.setLayoutParams(params);
        holder.mTv.setBackgroundColor(<span class="hljs-type">Color</span>.argb(<span class="hljs-number">255</span>, <span class="hljs-number">180</span> + mRandom.nextInt(<span class="hljs-number">60</span>) + <span class="hljs-number">30</span>, <span class="hljs-number">180</span> + mRandom.nextInt(<span class="hljs-number">60</span>) + <span class="hljs-number">30</span>, <span class="hljs-number">180</span> + mRandom.nextInt(<span class="hljs-number">60</span>) + <span class="hljs-number">30</span>));
        holder.mTv.setText(mData.get(position));
    }

    <span class="hljs-meta">@Override</span>
    public int getItemCount() {
        <span class="hljs-keyword">return</span> mData.size();
    }

    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyViewHolder</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RecyclerView</span>.<span class="hljs-title">ViewHolder</span> </span>{

        <span class="hljs-type">TextView</span> mTv;

        public <span class="hljs-type">MyViewHolder</span>(<span class="hljs-type">View</span> itemView) {
            <span class="hljs-keyword">super</span>(itemView);
            mTv = (<span class="hljs-type">TextView</span>) itemView.findViewById(android.<span class="hljs-type">R</span>.id.text1);
        }
    }

}</code></pre>
<p>经过上面对View.inflate()的第三个参数解析之后，这代码的问题一眼就能看出来了吧，没错，就是ViewGroup.LayoutParams params = holder.mTv.getLayoutParams();这行代码获取到的LayoutParams为空，不信？走一个。</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-f88230178d6b0a59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-f88230178d6b0a59.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>接下来理所当然的要让得到的LayoutParams不为空啦，所以将onCreateViewHolder()的代码修改如下：</p>
<pre class="hljs mel"><code class="mel">@Override
public MyViewHolder onCreateViewHolder(ViewGroup <span class="hljs-keyword">parent</span>, <span class="hljs-keyword">int</span> viewType) {
    <span class="hljs-comment">//这里使用的是安卓自带的文本控件布局</span>
    View itemView = View.inflate(<span class="hljs-keyword">parent</span>.getContext(), android.R.<span class="hljs-keyword">layout</span>.simple_list_item_1, <span class="hljs-keyword">parent</span>);
    <span class="hljs-keyword">return</span> new MyViewHolder(itemView);
}</code></pre>
<p>传入的ViewGroup parent不为null，所以肯定获取的LayoutParams不为空，但是又有一个问题，看报错。</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-b46de38b1c60918b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-b46de38b1c60918b.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>为什么会报这样的错呢？回看最终inflate()的四个步骤：</p>
<ol>
<li>通过使用XmlPullParser parser将xml布局文件转换成视图temp。</li>
<li>判断ViewGroup root对象是否为null，来决定要不要给temp设置LayoutParams。</li>
<li>判断boolean attachToRoot是否为true，来决定是否要把temp顺便加到ViewGroup root中。</li>
<li>最后返回视图temp。</li>
</ol>
<p>步骤2让条目获取的LayoutParams不为空没错，但是步骤3出问题了，当使用View.inflate(parent.getContext(), android.R.layout.simple_list_item_1, parent)传入parent后，boolean attachToRoot的取值就是为true，所以创建出来的条目会顺便添加到ViewGroup中（这里的ViewGroup就是RecyclerView），而RecyclerView本身就会自动将条目添加到自身，这样就添加了两次，故报错。那为什么attachToRoot的取值是true呢？再看View.inflate()的整个方法调用链：</p>
<pre class="hljs x86asm"><code class="x86asm">View.inflate() = 
    LayoutInflater.from(context)
<span class="hljs-meta">        .inflate</span>(resource, root)
<span class="hljs-meta">        .inflate</span>(resource, root, root != null)
<span class="hljs-meta">        .inflate</span>(parser, root, attachToRoot)</code></pre>
<p>boolean attachToRoot的取值取决于root（也就是parent）是否为空，这就是View.inflate()的瓶颈，它没法灵活的指定boolean attachToRoot的取值。这里我就是只是想让创建出来的视图能得到LayoutParams，但不添加到ViewGroup中，这样的要求可以通过LayoutInflater.from(context).inflate()来实现。所以下面将onCreateViewHolder()的代码修改如下：</p>
<pre class="hljs lasso"><code class="lasso">@Override
<span class="hljs-keyword">public</span> MyViewHolder onCreateViewHolder(ViewGroup <span class="hljs-keyword">parent</span>, int viewType) {
    View itemView = LayoutInflater.from(<span class="hljs-keyword">parent</span>.getContext()).inflate(android.R.layout.simple_list_item_1,<span class="hljs-keyword">parent</span>,<span class="hljs-literal">false</span>);
    ViewGroup.LayoutParams <span class="hljs-keyword">params</span> = itemView.getLayoutParams();
    <span class="hljs-keyword">Log</span>.e(<span class="hljs-string">"CSDN_LQR"</span>, <span class="hljs-string">"params == null : "</span> + (<span class="hljs-keyword">params</span> == <span class="hljs-built_in">null</span>));
    <span class="hljs-keyword">return</span> <span class="hljs-literal">new</span> MyViewHolder(itemView);
}</code></pre>
<p>代码中LayoutInflater.from(parent.getContext()).inflate(android.R.layout.simple_list_item_1,parent,false)传入了parent（即ViewGroup不为null），所以创建出来的视图可以得到LayoutParams，同时又指定attachToRoot的取值为false，即不添加到ViewGroup中。到这里，上面重覆添加子控件的问题就解决了，总结一下吧：</p>
<ul>
<li>View.inflate()第三个参数若不为null，则创建出来的视图一定能获得LayoutParams，反之，不一定。（下面会解释）</li>
<li>LayoutInflater.from(context).inflate()可以灵活的指定传入的ViewGroup是否为空来决定创建出来的视图能否获得LayoutParams，同时又可以指定attachToRoot的取值来决定创建出来的视图是否要添加到ViewGroup中。</li>
</ul>
<h1>三、小细节</h1>
<p><em>*上面已经将LayoutInflater的源码分析完毕，现在还有一个小问题，其实跟本文主题没多大关系，当作拓展来看吧。</em></p>
<p>前面说到，View.inflate()第三个参数若不为null，则创建出来的视图一定能获得LayoutParams，反之，不一定。这话怎么理解？</p>
<p>也就是说，即使View.inflate()第三个参数为null，创建出来的视图也有可能获得LayoutParams咯？是的，说到底，这个LayoutParams的有无，实际取决于条目本身是否有父控件，且看上面用到的simple_list_item_1布局：</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-0374f2c4185a463b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-0374f2c4185a463b.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<p>发现了吧，就一个TextView，没有父控件，那如果我给它加个父控件，同时使用最开始的方式也能顺利得到LayoutParams呢？代码如下： </p>
<pre class="hljs lasso"><code class="lasso">@Override
<span class="hljs-keyword">public</span> MyViewHolder onCreateViewHolder(ViewGroup <span class="hljs-keyword">parent</span>, int viewType) {
    View itemView = View.inflate(<span class="hljs-keyword">parent</span>.getContext(), R.layout.item_layout, <span class="hljs-built_in">null</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-literal">new</span> MyViewHolder(itemView);
}

@Override
<span class="hljs-keyword">public</span> <span class="hljs-literal">void</span> onBindViewHolder(MyViewHolder holder, int position) {
    ViewGroup.LayoutParams <span class="hljs-keyword">params</span> = holder.mTv.getLayoutParams();
    <span class="hljs-keyword">Log</span>.e(<span class="hljs-string">"CSDN_LQR"</span>, <span class="hljs-string">"params == null : "</span> + (<span class="hljs-keyword">params</span> == <span class="hljs-built_in">null</span>));
    <span class="hljs-params">...</span>
    控件设置
    <span class="hljs-params">...</span>
}</code></pre>
<p>item_layout的布局代码如下：</p>
<pre class="hljs xml"><code class="xml"><span class="php"><span class="hljs-meta">&lt;?</span>xml version=<span class="hljs-string">"1.0"</span> encoding=<span class="hljs-string">"utf-8"</span><span class="hljs-meta">?&gt;</span></span>
<span class="hljs-tag">&lt;<span class="hljs-name">LinearLayout</span> <span class="hljs-attr">xmlns:android</span>=<span class="hljs-string">"http://schemas.android.com/apk/res/android"</span>
              <span class="hljs-attr">android:layout_width</span>=<span class="hljs-string">"match_parent"</span>
              <span class="hljs-attr">android:layout_height</span>=<span class="hljs-string">"match_parent"</span>
              <span class="hljs-attr">android:orientation</span>=<span class="hljs-string">"vertical"</span>&gt;</span>

    <span class="hljs-tag">&lt;<span class="hljs-name">TextView</span>
        <span class="hljs-attr">android:id</span>=<span class="hljs-string">"@android:id/text1"</span>
        <span class="hljs-attr">android:layout_width</span>=<span class="hljs-string">"match_parent"</span>
        <span class="hljs-attr">android:layout_height</span>=<span class="hljs-string">"wrap_content"</span>
        <span class="hljs-attr">android:gravity</span>=<span class="hljs-string">"center_vertical"</span>
        <span class="hljs-attr">android:minHeight</span>=<span class="hljs-string">"?android:attr/listPreferredItemHeightSmall"</span>
        <span class="hljs-attr">android:textAppearance</span>=<span class="hljs-string">"?android:attr/textAppearanceListItemSmall"</span>/&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">LinearLayout</span>&gt;</span></code></pre>
<p>运行，果然可以获得LayoutParams，打印结果如下：</p>
<div class="image-package">
<img src="//upload-images.jianshu.io/upload_images/4050443-fc3087743deeb567.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/4050443-fc3087743deeb567.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption"></div>
</div>
<h1>四、最后</h1>
<p>本人也是头次写类分析型的文章，如描述有误，请不吝赐教，同时还请各位看客多担待，指出后本人会尽快修改，谢谢。</p>

        </div>
        <!--  -->

        <div class="show-foot">
          <a class="notebook" href="/nb/11969170">
            <i class="iconfont ic-search-notebook"></i> <span>Android高级UI</span>
</a>          <div class="copyright" data-toggle="tooltip" data-html="true" data-original-title="转载请联系作者获得授权，并标注“简书作者”。">
            © 著作权归作者所有
          </div>
          <div class="modal-wrap" data-report-note="">
            <a id="report-modal">举报文章</a>
          </div>
        </div>
    </div>