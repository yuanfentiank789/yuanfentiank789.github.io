---
layout: post
title:  "Android 5.0(Lollipop)中的SurfaceTexture，TextureView"
date:   2016-07-18 1:05:00
catalog:  true
tags:
    - SurfaceTexture
    - TextureView
   
---


  <div class="content_l">
    <div class="nzpath">您当前位置：<a href="http://www.wfuyu.com/">首页</a> &gt; <a href="http://www.wfuyu.com/phpcms/">php开源</a> &gt; <a href="http://www.wfuyu.com/technology/">综合技术</a> &gt; Android 5.0(Lollipop)中的SurfaceTexture，TextureView, SurfaceView和GLSurfaceView
</div>
  <div class="font_content">
  
   <h1 class="h1">Android 5.0(Lollipop)中的SurfaceTexture，TextureView, SurfaceView和GLSurfaceView</h1>
   <div class="newsbanner">
         <div class="laiyuan">来源：程序员人生&nbsp;&nbsp; 发布时间：2015-03-23 07:51:29 阅读次数：4508次</div>
      </div> 
     

<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">SurfaceView, GLSurfaceView, SurfaceTexture和TextureView是Android当中名字比较绕，关系又比较密切的几个类。本文基于Android 5.0(Lollipop)的代码理1下它们的基本原理，联系与区分。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<p style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><strong>SurfaceView</strong>从Android 1.0(API level 1)时就有 。它继承自类View，因此它本质上是1个View。但与普通View不同的是，它有自己的Surface。我们知道，1般的Activity包括的多个View会组成View hierachy的树形结构，只有最顶层的DecorView，也就是根结点视图，才是对WMS可见的。这个DecorView在WMS中有1个对应的WindowState。相应地，在SF中对应的Layer。而SurfaceView自带1个Surface，这个Surface在WMS中有自己对应的WindowState，在SF中也会有自己的Layer。以下图所示：</p>
<p style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150304164219975.png" alt=""><br>
</p>
<p style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"></p>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">也就是说，虽然在App端它仍在View hierachy中，但在Server端（WMS和SF）中，它与宿主窗口是分离的。这样的好处是对这个Surface的渲染可以放到单独线程去做，渲染时可以有自己的GL context。这对1些游戏、视频等性能相干的利用非常有益，由于它不会影响主线程对事件的响应。但它也有缺点，由于这个Surface不在View hierachy中，它的显示也不受View的属性控制，所以不能进行平移，缩放等变换，也不能放在其它ViewGroup中，1些View中的特性也没法使用。</span></div>
<br>
<p></p>
<p style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"></p>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><strong>GLSurfaceView</strong>从Android 1.5(API level 3)开始加入，作为SurfaceView的补充。它可以看做是SurfaceView的1种典型使用模式。在SurfaceView的基础上，它加入了EGL的管理，并自带了渲染线程。另外它定义了用户需要实现的Render接口，提供了用Strategy pattern更改具体Render行动的灵活性。作为GLSurfaceView的Client，只需要将实现了渲染函数的Renderer的实现类设置给GLSurfaceView便可。如：</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><div class="w_dama2">public class TriangleActivity extends Activity {
    protected void onCreate(Bundle savedInstanceState) {
        mGLView = new GLSurfaceView(this);
        mGLView.setRenderer(new RendererImpl(this));</div><span style="">相干类图以下。其中SurfaceView中的SurfaceHolder主要是提供了1坨操作Surface的接口。GLSurfaceView中的EglHelper和GLThread分别实现了上面提到的管理EGL环境和渲染线程的工作。GLSurfaceView的使用者需要实现Renderer接口。</span></div>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><span style=""><img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305081903285.png" alt=""><br>
</span></div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><span style=""><br>
</span></div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><span style=""></span>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><strong>SurfaceTexture</strong></span><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">从Android 3.0(API level 11)加入。和SurfaceView不同的是，它对图象流的处理其实不直接显示，而是转为GL外部纹理，因此可用于图象流数据的2次处理（如Camera滤镜，桌面殊效等）。比如Camera的预览数据，变成纹理后可以交给GLSurfaceView直接显示，也能够通过SurfaceTexture交给TextureView作为View
 heirachy中的1个硬件加速层来显示。首先，SurfaceTexture从图象流（来自Camera预览，视频解码，GL绘制场景等）中取得帧数据，当调用updateTexImage()时，根据内容流中最近的图象更新SurfaceTexture对应的GL纹理对象，接下来，就能够像操作普通GL纹理1样操作它了。从下面的类图中可以看出，它核心管理着1个BufferQueue的Consumer和Producer两端。Producer端用于内容流的源输出数据，Consumer端用于拿GraphicBuffer并生成纹理。SurfaceTexture.OnFrameAvailableListener用于让SurfaceTexture的使用者知道有新数据到来。JNISurfaceTextureContext是OnFrameAvailableListener从Native到Java的JNI跳板。其中SurfaceTexture中的attachToGLContext()和detachToGLContext()可让多个GL
 context同享同1个内容源。</span></div>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305082032854.png" alt=""><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><span style=""><br>
</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">Android 5.0中将BufferQueue的核心功能分离出来，放在BufferQueueCore这个类中。BufferQueueProducer和BufferQueueConsumer分别是它的生产者和消费者实现基类（分别实现了IGraphicBufferProducer和IGraphicBufferConsumer接口）。它们都是由BufferQueue的静态函数createBufferQueue()来创建的。Surface是生产者真个实现类，提供dequeueBuffer/queueBuffer等硬件渲染接口，和lockCanvas/unlockCanvasAndPost等软件渲染接口，使内容流的源可以往BufferQueue中填graphic
 buffer。GLConsumer继承自ConsumerBase，是消费者真个实现类。它在基类的基础上添加了GL相干的操作，如将graphic buffer中的内容转为GL纹理等操作。到此，以SurfaceTexture为中心的1个pipeline大体是这样的：</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305082434209.png" alt=""><br>
</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"></span>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><strong>TextureView</strong></span><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">在4.0(API level 14)中引入。它可以将内容流直接投影到View中，可以用于实现Live preview等功能。和SurfaceView不同，它不会在WMS中单独创建窗口，而是作为View
 hierachy中的1个普通View，因此可以和其它普通View1样进行移动，旋转，缩放，动画等变化。值得注意的是TextureView必须在硬件加速的窗口中。它显示的内容流数据可以来自App进程或是远端进程。从类图中可以看到，TextureView继承自View，它与其它的View1样在View hierachy中管理与绘制。TextureView重载了draw()方法，其中主要把SurfaceTexture中收到的图象数据作为纹理更新到对应的HardwareLayer中。SurfaceTexture.OnFrameAvailableListener用于通知TextureView内容流有新图象到来。SurfaceTextureListener接口用于让TextureView的使用者知道SurfaceTexture已准备好，这样就能够把SurfaceTexture交给相应的内容源。Surface为BufferQueue的Producer接口实现类，使生产者可以通过它的软件或硬件渲染接口为SurfaceTexture内部的BufferQueue提供graphic
 buffer。</span></div>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305082558158.jpg" alt=""><br>
</div>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">下面以VideoDumpView.java（位于/frameworks/base/media/tests/MediaDump/src/com/android/mediadump/）为例分析下SurfaceTexture的使用。这个例子的效果是从MediaPlayer中拿到视频帧，然后显示在屏幕上，接着把屏幕上的内容dump到指定文件中。由于SurfaceTexture本身只产生纹理，所以这里还需要GLSurfaceView配合来做最后的渲染输出。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">首先，VideoDumpView是GLSurfaceView的继承类。在构造函数VideoDumpView()中会创建VideoDumpRenderer，也就是GLSurfaceView.Renderer的实例，然后调setRenderer()将之设成GLSurfaceView的Renderer。</div>
</div>
<div class="w_dama2">109    public VideoDumpView(Context context) {
...
116        mRenderer = new VideoDumpRenderer(context);
117        setRenderer(mRenderer);
118    }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">随后，GLSurfaceView中的GLThread启动，创建EGL环境后回调VideoDumpRenderer中的onSurfaceCreated()。</span></div>
<div class="w_dama2">519        public void onSurfaceCreated(GL10 glUnused, EGLConfig config) {
...
551            // Create our texture. This has to be done each time the surface is created.
552            int[] textures = new int[1];
553            GLES20.glGenTextures(1, textures, 0);
554
555            mTextureID = textures[0];
556            GLES20.glBindTexture(GL_TEXTURE_EXTERNAL_OES, mTextureID);
...
575            mSurface = new SurfaceTexture(mTextureID);
576            mSurface.setOnFrameAvailableListener(this);
577
578            Surface surface = new Surface(mSurface);
579            mMediaPlayer.setSurface(surface);</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">这里，首先通过GLES创建GL的外部纹理。外部纹理说明它的真正内容是放在ion分配出来的系统物理内存中，而不是GPU中，GPU中只是保护了其元数据。接着根据前面创建的GL纹理对象创建SurfaceTexture。流程以下：</span></div>
<p><img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305082941877.jpg" alt=""></p>
<p></p>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">SurfaceTexture的参数为GLES接口函数glGenTexture()得到的纹理对象id。在初始化函数SurfaceTexture_init()中，先创建GLConsumer和相应的BufferQueue，再将它们的指针通过JNI放到SurfaceTexture的Java层对象成员中。</span></div>
<div class="w_dama2">230static void SurfaceTexture_init(JNIEnv* env, jobject thiz, jboolean isDetached,
231        jint texName, jboolean singleBufferMode, jobject weakThiz)
232{
...
235    BufferQueue::createBufferQueue(&amp;producer, &amp;consumer);
...
242    sp&lt;GLConsumer&gt; surfaceTexture;
243    if (isDetached) {
244        surfaceTexture = new GLConsumer(consumer, GL_TEXTURE_EXTERNAL_OES,
245                true, true);
246    } else {
247        surfaceTexture = new GLConsumer(consumer, texName,
248                GL_TEXTURE_EXTERNAL_OES, true, true);
249    }
...
256    SurfaceTexture_setSurfaceTexture(env, thiz, surfaceTexture);
257    SurfaceTexture_setProducer(env, thiz, producer);
...
266    sp&lt;JNISurfaceTextureContext&gt; ctx(new JNISurfaceTextureContext(env, weakThiz,
267            clazz));
268    surfaceTexture-&gt;setFrameAvailableListener(ctx);
269    SurfaceTexture_setFrameAvailableListener(env, thiz, ctx);</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">由于直接的Listener在Java层，而触发者在Native层，因此需要从Native层回调到Java层。这里通过JNISurfaceTextureContext当了跳板。JNISurfaceTextureContext的onFrameAvailable()起到了Native和Java的桥接作用：</span></div>
<div class="w_dama2">180void JNISurfaceTextureContext::onFrameAvailable()
...
184        env-&gt;CallStaticVoidMethod(mClazz, fields.postEvent, mWeakThiz);</div>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">其中的fields.postEvent早在SurfaceTexture_classInit()中被初始化为SurfaceTexture的postEventFromNative()函数。这个函数往所在线程的消息队列中放入消息，异步调用VideoDumpRenderer的onFrameAvailable()函数，通知VideoDumpRenderer有新的数据到来。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">回到onSurfaceCreated()，接下来创建供外部生产者使用的Surface类。Surface的构造函数之1带有参数SurfaceTexture。</div>
</div>
<div class="w_dama2">133    public Surface(SurfaceTexture surfaceTexture) {
...
140            setNativeObjectLocked(nativeCreateFromSurfaceTexture(surfaceTexture));</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">它实际上是把SurfaceTexture中创建的BufferQueue的Producer接口实现类拿出来后创建了相应的Surface类。</span></div>
<div class="w_dama2">135static jlong nativeCreateFromSurfaceTexture(JNIEnv* env, jclass clazz,
136        jobject surfaceTextureObj) {
137    sp&lt;IGraphicBufferProducer&gt; producer(SurfaceTexture_getProducer(env, surfaceTextureObj));
...
144    sp&lt;Surface&gt; surface(new Surface(producer, true));</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">这样，Surface为BufferQueue的Producer端，SurfaceTexture中的GLConsumer为BufferQueue的Consumer端。当通过Surface绘制时，SurfaceTexture可以通过updateTexImage()来将绘制结果绑定到GL的纹理中。</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</span></div>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">回到onSurfaceCreated()函数，接下来调用setOnFrameAvailableListener()函数将VideoDumpRenderer（实现SurfaceTexture.OnFrameAvailableListener接口）作为SurfaceTexture的Listener，由于它要监听内容流上是不是有新数据。接着将SurfaceTexture传给MediaPlayer，由于这里MediaPlayer是生产者，SurfaceTexture是消费者。后者要接收前者输出的Video
 frame。这样，就通过Observer pattern建立起了1条通知链：MediaPlayer -&gt; SurfaceTexture -&gt; VideDumpRenderer。在onFrameAvailable()回调函数中，将updateSurface标志设为true，表示有新的图象到来，需要更新Surface了。为毛不在这儿马上更新纹理呢，由于当前可能不在渲染线程。SurfaceTexture对象可以在任意线程被创建（回调也会在该线程被调用），但updateTexImage()只能在含有纹理对象的GL
 context所在线程中被调用。因此1般情况下回调中不能直接调用updateTexImage()。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">与此同时，GLSurfaceView中的GLThread也在运行，它会调用到VideoDumpRenderer的绘制函数onDrawFrame()。</div>
</div>
<div class="w_dama2">372        public void onDrawFrame(GL10 glUnused) {
...
377                if (updateSurface) {
...
380                    mSurface.updateTexImage();
381                    mSurface.getTransformMatrix(mSTMatrix);
382                    updateSurface = false;
...
394            // Activate the texture.
395            GLES20.glActiveTexture(GLES20.GL_TEXTURE0);
396            GLES20.glBindTexture(GL_TEXTURE_EXTERNAL_OES, mTextureID);
...
421            // Draw a rectangle and render the video frame as a texture on it.
422            GLES20.glDrawArrays(GLES20.GL_TRIANGLE_STRIP, 0, 4);
...
429                DumpToFile(frameNumber);</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">这里，通过SurfaceTexture的updateTexImage()将内容流中的新图象转成GL中的纹理，再进行坐标转换。绑定刚生成的纹理，画到屏幕上。全部流程以下：</span></div>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305083843743.jpg" alt=""><br>
<p></p>
<p></p>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">最后onDrawFrame()调用DumpToFile()将屏幕上的内容倒到文件中。在DumpToFile()中，先用glReadPixels()从屏幕中把像素数据存到Buffer中，然后用FileOutputStream输出到文件。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">上面讲了SurfaceTexture，下面看看TextureView是如何工作的。还是从例子着手，Android的关于TextureView的官方文档(http://developer.android.com/reference/android/view/TextureView.html)给了1个简洁的例子LiveCameraActivity。它它可以将Camera中的内容放在View中进行显示。在onCreate()函数中首先创建TextureView，再将Activity(实现了TextureView.SurfaceTextureListener接口)传给TextureView，用于监听SurfaceTexture准备好的信号。</div>
</div>
<div class="w_dama2">      protected void onCreate(Bundle savedInstanceState) {
          ...
          mTextureView = new TextureView(this);
          mTextureView.setSurfaceTextureListener(this);
          ...
      }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">TextureView的构造函数其实不做主要的初始化工作。主要的初始化工作是在getHardwareLayer()中，而这个函数是在其基类View的draw()中调用。TextureView重载了这个函数：</span></div>
<div class="w_dama2">348    HardwareLayer getHardwareLayer() {
...
358            mLayer = mAttachInfo.mHardwareRenderer.createTextureLayer();
359            if (!mUpdateSurface) {
360                // Create a new SurfaceTexture for the layer.
361                mSurface = new SurfaceTexture(false);
362                mLayer.setSurfaceTexture(mSurface);
363            }
364            mSurface.setDefaultBufferSize(getWidth(), getHeight());
365            nCreateNativeWindow(mSurface);
366
367            mSurface.setOnFrameAvailableListener(mUpdateListener, mAttachInfo.mHandler);
368
369            if (mListener != null &amp;&amp; !mUpdateSurface) {
370                mListener.onSurfaceTextureAvailable(mSurface, getWidth(), getHeight());
371            }
...
390        applyUpdate();
391        applyTransformMatrix();
392
393        return mLayer;
394    }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">由于TextureView是硬件加速层（类型为LAYER_TYPE_HARDWARE），它首先通过HardwareRenderer创建相应的HardwareLayer类，放在mLayer成员中。然后创建SurfaceTexture类，具体流程见前文。以后将HardwareLayer与SurfaceTexture做绑定。接着调用Native函数nCreateNativeWindow，它通过SurfaceTexture中的BufferQueueProducer创建Surface类。注意Surface实现了ANativeWindow接口，这意味着它可以作为EGL
 Surface传给EGL接口从而进行硬件绘制。然后setOnFrameAvailableListener()将监听者mUpdateListener注册到SurfaceTexture。这样，当内容流上有新的图象到来，mUpdateListener的onFrameAvailable()就会被调用。然后需要调用注册在TextureView中的SurfaceTextureListener的onSurfaceTextureAvailable()回调函数，通知TextureView的使用者SurfaceTexture已就绪。全部流程大体以下：</span></div>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305085308993.jpg" alt=""><br>
<p></p>
<p></p>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">注意这里这里为TextureView创建了DeferredLayerUpdater，而不是像Android 4.4(Kitkat)中返回GLES20TextureLayer。由于Android 5.0(Lollipop)中在App端分离出了渲染线程，并将渲染工作放到该线程中。这个线程还能接收VSync信号，因此它还能自己处理动画。事实上，这里DeferredLayerUpdater的创建就是通过同步方式在渲染线程中做的。DeferredLayerUpdater，顾名思义，就是将Layer的更新要求先记录在这，当渲染线程真正要画的时候，再进行真实的操作。其中的setSurfaceTexture()会调用HardwareLayer的Native函数nSetSurfaceTexture()将SurfaceTexture中的surfaceTexture成员（类型为GLConsumer）传给DeferredLayerUpdater，这样以后要更新纹理时DeferredLayerUpdater就知道从哪里更新了。</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">前面提到初始化中会调用onSurfaceTextureAvailable()这个回调函数。在它的实现中，TextureView的使用者就能够将准备好的SurfaceTexture传给数据源模块，供数据源输出之用。如：</div>
</div>
<div class="w_dama2">      public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
          mCamera = Camera.open();
              ...
              mCamera.setPreviewTexture(surface);
              mCamera.startPreview();
              ...
      }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">看1下setPreviewTexture()的实现，其中把SurfaceTexture中初始化时创建的GraphicBufferProducer拿出来传给Camera模块。</span></div>
<div class="w_dama2">576static void android_hardware_Camera_setPreviewTexture(JNIEnv *env,
577        jobject thiz, jobject jSurfaceTexture)
...
585        producer = SurfaceTexture_getProducer(env, jSurfaceTexture);
...
594    if (camera-&gt;setPreviewTarget(producer) != NO_ERROR) {</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">到这里，1切都初始化地差不多了。接下来当内容流有新图象可用，TextureView会被通知到（通过SurfaceTexture.OnFrameAvailableListener接口）。SurfaceTexture.OnFrameAvailableListener是SurfaceTexture有新内容来时的回调接口。TextureView中的mUpdateListener实现了该接口：</span></div>
<div class="w_dama2">755        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
756            updateLayer();
757            invalidate();
758        }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">可以看到其中会调用updateLayer()函数，然后通过invalidate()函数申请更新UI。updateLayer()会设置mUpdateLayer标志位。这样，当下次VSync到来时，Choreographer通知App通太重绘View hierachy。在UI重绘函数performTranversals()中，作为View hierachy的1份子，TextureView的draw()函数被调用，其中便会相继调用applyUpdate()和HardwareLayer的updateSurfaceTexture()函数。</span></div>
<div class="w_dama2">138    public void updateSurfaceTexture() {
139        nUpdateSurfaceTexture(mFinalizer.get());
140        mRenderer.pushLayerUpdate(this);
141    }</div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">updateSurfaceTexture()实际通过JNI调用到android_view_HardwareLayer_updateSurfaceTexture()函数。在其中会设置相应DeferredLayerUpdater的标志位mUpdateTexImage，它表示在渲染线程中需要更新该层的纹理。</span></div>
<br>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305084333832.jpg" alt=""><br>
<br>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">前面提到，Android 5.0引入了渲染线程，它是1个更大的topic，超越本文范围，这里只说相干的部份。作为背景知识，下面只画出了相干的类。可以看到，ThreadedRenderer作为新的HardwareRenderer替换了Android 4.4中的Gl20Renderer。其中比较关键的是RenderProxy类，需要让渲染线程干活时就通过这个类往渲染线程发任务。RenderProxy中指向的RenderThread就是渲染线程的主体了，其中的threadLoop()函数是主循环，大多数时间它会poll在线程的Looper上等待，当有同步要求（或VSync信号）过来，它会被唤醒，然后处理TaskQueue中的任务。TaskQueue是RenderTask的队列，RenderTask代表1个渲染线程中的任务。如DrawFrameTask就是RenderTask的继承类之1，它主要用于渲染当前帧。而DrawFrameTask中的DeferredLayerUpdater集合就寄存着之前对硬件加速层的更新操作申请。</span></div>
<img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305085434529.jpg" alt="">
<p></p>
<p></p>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">当主线程准备好渲染数据后，会以同步方式让渲染线程完成渲染工作。其中会先调用processLayerUpdate()更新所有硬件加速层中的属性，继而调用到DeferredLayerUpdater的apply()函数，其中检测到标志位mUpdateTexImage被置位，因而会调用doUpdateTexImage()真正更新GL纹理和转换坐标。</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/uploadfile/cj/20150307/20150305085604765.jpg" alt=""><br>
</span></div>
<div><span style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"></span>
<div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px"><br>
</div>
<div style="font-family:微软雅黑; orphans:2; widows:2; font-size:14px">最后，总结下这几者的区分和联系。简单地说，SurfaceView是1个有自己Surface的View。它的渲染可以放在单独线程而不是主线程中。其缺点是不能做变形和动画。SurfaceTexture可以用作非直接输出的内容流，这样就提供2次处理的机会。与SurfaceView直接输出相比，这样会有若干帧的延迟。同时，由于它本身管理BufferQueue，因此内存消耗也会略微大1些。TextureView是1个可以把内容流作为外部纹理输出在上面的View。它本身需要是1个硬件加速层。事实上TextureView本身也包括了SurfaceTexture。它与SurfaceView+SurfaceTexture组合相比可以完成类似的功能（即把内容流上的图象转成纹理，然后输出）。区分在于TextureView是在View
 hierachy中做绘制，因此1般它是在主线程上做的（在Android 5.0引入渲染线程后，它是在渲染线程中做的）。而SurfaceView+SurfaceTexture在单独的Surface上做绘制，可以是用户提供的线程，而不是系统的主线程或是渲染线程。另外，与TextureView相比，它还有个好处是可以用Hardware overlay进行显示。</div>
<br>
</div>
<br>
</div>
<br>
<br>
<br>
<p></p>
<br>
<br>
     <div style=" height:300px; width:600px; margin:0 auto; text-align:center; font: bold 14px/20px;">
 生活不易，码农辛苦<br>
 如果您觉得本网站对您的学习有所帮助,可以手机扫描二维码进行捐赠<br>
  <img data-bd-imgshare-binded="1" src="http://www.wfuyu.com/skin/images/weixin.png" alt="程序员人生" height="200px" width="200px">
</div>
     </div>
     
     
        
  
  </div>
