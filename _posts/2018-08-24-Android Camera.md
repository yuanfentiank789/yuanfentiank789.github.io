---
layout: post
title:  "Camera使用记录"
date:   2018-08-24 1:05:00
catalog:  true
tags:
    - Camera


---

<div class="post-content" id="post-content" itemprop="postContent">
            <h2 id="前言"><a href="#前言" class="headerlink" title="前言"></a>前言</h2><p>首先，本文主要以 <strong>android.hardware.Camera</strong> 包来叙述内容，其实Google官方已经把其作为Deprecated的了，但由于新的包 <strong>android.hardware.camera2 </strong>需要最低API level 21，即需要Android 5.0以上，且国内很多厂商对这个接口的支持并不好，因此暂不考虑（主要内容其实变化不大）<br>有兴趣的同学，可以看 <a href="https://github.com/google/cameraview" target="_blank" rel="external">https://github.com/google/cameraview</a> ，Google提供的一个CameraView，可以自动判断使用哪个API~</p>
<h2 id="开发流程"><a href="#开发流程" class="headerlink" title="开发流程"></a>开发流程</h2><p><a href="https://developer.android.com/reference/android/hardware/Camera.html" target="_blank" rel="external">https://developer.android.com/reference/android/hardware/Camera.html</a><br>官方引导文档如上，大致开发流程可以按照上述文档来，本文主要内容讲拍照相关，摄像相关暂不涉猎。</p>
<h3 id="API选择"><a href="#API选择" class="headerlink" title="API选择"></a>API选择</h3><figure class="image-bubble">
                <div class="img-lightbox">
                    <div class="overlay"></div>
                    <img src="https://ws2.sinaimg.cn/large/006tKfTcgy1fmcn3b6mrjj30k20agmxb.jpg" alt="" title="">
                </div>
                <div class="image-caption"></div>
            </figure>
<p>首先先要进行API选择，Google共提供两种控制摄像头的API，新的API是Camera2… 但是点评美团的minSdk是16，估计短期内都无法升到21，因此使用Camera1进行，当然也可以直接使用google的这个CameraView，根据SDK VERSION使用不同的API。</p>
<figure class="highlight kotlin"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line"><span class="keyword">if</span> (Build.VERSION.SDK_INT &lt; <span class="number">21</span>) &#123;            </div><div class="line">		mImpl = new Camera1(mCallbacks, preview);        </div><div class="line">&#125; <span class="keyword">else</span> <span class="keyword">if</span> (Build.VERSION.SDK_INT &lt; <span class="number">23</span>) &#123;            </div><div class="line">		mImpl = new Camera2(mCallbacks, preview, context);        </div><div class="line">&#125; <span class="keyword">else</span> &#123;            </div><div class="line">		mImpl = new Camera2Api23(mCallbacks, preview, context);        </div><div class="line">&#125;</div></pre></td></tr></table></figure>
<p>本着嫌麻烦的原则，放弃了该方法，主要原因有2：</p>
<ol>
<li>测试需要测试3种不同Android版本的手机，确保每个分支都覆盖到，条件不允许</li>
<li>点评、美团客户端对第三方包的引入都有限制，非com.dianping or com.meituan的包都需要申请（support包中的gridview都有问题，更不必说这个camera了）而自己粘的话，会引入不少代码，因此放弃</li>
</ol>
<p>下面的讲解均按照Camera1来讲述，Camera2会有些不同，仅供参考</p>
<h2 id="Camera工作流程"><a href="#Camera工作流程" class="headerlink" title="Camera工作流程"></a>Camera工作流程</h2><h3 id="SurfaceView-vs-TextureView"><a href="#SurfaceView-vs-TextureView" class="headerlink" title="SurfaceView vs TextureView"></a>SurfaceView vs TextureView</h3><p>代码核心的元素有2个，一个是Camera用于控制及获取摄像头信息，另一个是用于展示预览效果的View，可以使用TextureView or SurfaceView。这两个View的绘制都是在独立的线程中，因此在预览摄像头的过程中并不会阻塞UI线程，并且SurfaceView使用一个独立的窗口，无法对其使用平移、缩放等动画，而TextureView和普通的View在同一个窗口下，可以使用动画。详细的比较可以看<a href="https://github.com/crosswalk-project/crosswalk-website/wiki/Android-SurfaceView-vs-TextureView" target="_blank" rel="external">https://github.com/crosswalk-project/crosswalk-website/wiki/Android-SurfaceView-vs-TextureView</a>，这里介绍了很多SurfaceView与TextureView的对比。RD可以根据需求选用相应的View预览摄像头的内容。主要分别以下：</p>
<ol>
<li>SurfaceView不能执行View的动画，TextureView可以</li>
<li>多个SurfaceView不能叠加，TextureView可以</li>
<li>TextureView必须在硬件加速的窗口中</li>
<li>TextureView消耗的内存比SurfaceView大</li>
</ol>
<p>综上所述，在美团8.9海外拍照翻译中，使用SurfaceView进行承载摄像头数据。</p>
<h3 id="Camera初始化"><a href="#Camera初始化" class="headerlink" title="Camera初始化"></a>Camera初始化</h3><p>首先，在SurfaceView的SurfaceCreated回调中进行摄像头的初始化，设置相关参数。</p>
<figure class="highlight kotlin"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div></pre></td><td class="code"><pre><div class="line"><span class="comment">// 选择摄像头，使用后置摄像头</span></div><div class="line">int numberOfCameras = Camera.getNumberOfCameras();</div><div class="line"><span class="keyword">for</span> (int i = <span class="number">0</span>; i &lt; numberOfCameras; i++) &#123;</div><div class="line">    Camera.CameraInfo info = new Camera.CameraInfo();</div><div class="line">    Camera.getCameraInfo(i, info);</div><div class="line">    <span class="keyword">if</span> (info.facing == Camera.CameraInfo.CAMERA_FACING_BACK) &#123;</div><div class="line">        mCameraId = i;</div><div class="line">        <span class="keyword">break</span>;</div><div class="line">    &#125;</div><div class="line">&#125;</div><div class="line"></div><div class="line">mCamera = Camera.<span class="keyword">open</span>(mCameraId);</div><div class="line">mCamera.setPreviewDisplay(mSurfaceView.getHolder());</div><div class="line">mCamera.startPreview();</div></pre></td></tr></table></figure>
<p>摄像头可以有多个（前置 &amp; 后置），根据需求选择相应的摄像头，并通过open(cameraId)方法获取并打开相应摄像头（默认不传参打开后置，但很怕有特的手机 or Rom默认前置摄像头的…）<br>对于使用SurfaceView承载预览数据的，通过使用setPreviewDisplay方法，设置预览所用的内容，这时通过startPreview就会发现，SurfaceView的部分会展示摄像头的内容了~</p>
<h3 id="Camera参数配置"><a href="#Camera参数配置" class="headerlink" title="Camera参数配置"></a>Camera参数配置</h3><p>如果你是直接按照上述代码开发，有很大概率你会踩到深坑~ 主要以下N个（一个个都是泪啊）：</p>
<ul>
<li>你会发现，你预览的窗口是歪的… 即你是竖着拍，但是SurfaceView中的预览是横着的…这个原因是摄像头的安装方向和手机垂直方向不一致，而是使用手机的水平方向… 因此，需要设置摄像头的方向，这时候又会引入另一个问题，就是摄像头的安装方向也可能不一样，大部分手机是旋转90°，但有些手机是需要旋转270°… 如坑爹的Nexus 5X… 因此这个旋转角度还不能写死，需要通过获取CameraInfo获取方向的不同来设置不同的角度~   (╬￣皿￣)=○</li>
</ul>
<figure class="highlight kotlin"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div><div class="line">15</div><div class="line">16</div><div class="line">17</div><div class="line">18</div><div class="line">19</div><div class="line">20</div><div class="line">21</div><div class="line">22</div><div class="line">23</div><div class="line">24</div><div class="line">25</div><div class="line">26</div><div class="line">27</div><div class="line">28</div></pre></td><td class="code"><pre><div class="line">android.hardware.Camera.CameraInfo info = new android.hardware.Camera.CameraInfo();</div><div class="line">android.hardware.Camera.getCameraInfo(cameraId, info);</div><div class="line">int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();</div><div class="line">int degrees = <span class="number">0</span>;</div><div class="line">switch (rotation) &#123;</div><div class="line">    case Surface.ROTATION_0:</div><div class="line">        degrees = <span class="number">0</span>;</div><div class="line">        <span class="keyword">break</span>;</div><div class="line">    case Surface.ROTATION_90:</div><div class="line">        degrees = <span class="number">90</span>;</div><div class="line">        <span class="keyword">break</span>;</div><div class="line">    case Surface.ROTATION_180:</div><div class="line">        degrees = <span class="number">180</span>;</div><div class="line">        <span class="keyword">break</span>;</div><div class="line">    case Surface.ROTATION_270:</div><div class="line">        degrees = <span class="number">270</span>;</div><div class="line">        <span class="keyword">break</span>;</div><div class="line">&#125;</div><div class="line"></div><div class="line">int result;</div><div class="line"><span class="keyword">if</span> (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) &#123;</div><div class="line">    result = (info.orientation + degrees) % <span class="number">360</span>;</div><div class="line">    result = (<span class="number">360</span> - result) % <span class="number">360</span>;      <span class="comment">// compensate the mirror</span></div><div class="line">&#125; <span class="keyword">else</span> &#123;</div><div class="line">    <span class="comment">// back-facing</span></div><div class="line">    result = (info.orientation - degrees + <span class="number">360</span>) % <span class="number">360</span>;</div><div class="line">&#125;</div><div class="line">mCamera.setDisplayOrientation(result);</div></pre></td></tr></table></figure>
<ul>
<li><p>预览、拍照分辨率很不清晰造成这个情况的主要原因有二，一是预览和拍照的分辨率需要设置且需要单独设置！二是预览和拍照时需要进行对焦。首先针对第一个问题踩的坑进行说明，主要需要注意以下两点：</p>
<ul>
<li><p>分辨率单独设置预览分辨率 和 拍照分辨率是两个不同的属性，需要单独设置。而且，每个摄像头只能设置成其支持的预览和拍照分辨率，不是可以随便任意塞值的。比如你的surfaceView是1050x1564这种诡异的分辨率，那就<strong>只能</strong>（没经过太多调研，如果有其他好方案，谢谢告知）手动去截断 or 自动拉伸。同时，摄像头支持的 拍照分辨率 和 预览分辨率 很可能不相符，代码如下：</p>
<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div><div class="line">15</div><div class="line">16</div><div class="line">17</div><div class="line">18</div><div class="line">19</div><div class="line">20</div><div class="line">21</div><div class="line">22</div><div class="line">23</div><div class="line">24</div><div class="line">25</div><div class="line">26</div><div class="line">27</div><div class="line">28</div><div class="line">29</div><div class="line">30</div><div class="line">31</div><div class="line">32</div><div class="line">33</div><div class="line">34</div><div class="line">35</div><div class="line">36</div><div class="line">37</div><div class="line">38</div></pre></td><td class="code"><pre><div class="line"><span class="comment">// 获取支持分辨率</span></div><div class="line">mCamera.getParameters().getSupportedPreviewSizes();</div><div class="line">mCamera.getParameters().getSupportedPictureSizes();</div><div class="line"></div><div class="line"><span class="comment">/**</span></div><div class="line"><span class="comment">    * 获取最佳预览分辨率</span></div><div class="line"><span class="comment">    *</span></div><div class="line"><span class="comment">    * <span class="doctag">@param</span> sizeList    摄像头支持分辨率列表</span></div><div class="line"><span class="comment">    * <span class="doctag">@return</span> 适当的分辨率，如果没有符合逻辑的，则选用最大的</span></div><div class="line"><span class="comment">    */</span></div><div class="line">   <span class="keyword">public</span> <span class="keyword">static</span> Camera.<span class="function">Size <span class="title">getFitSize</span><span class="params">(List&lt;Camera.Size&gt; sizeList)</span> </span>&#123;</div><div class="line"></div><div class="line">       <span class="comment">// 首先，将摄像头支持的分辨率 降序排序</span></div><div class="line">       <span class="keyword">if</span> (CollectionUtils.isEmpty(sizeList)) &#123;</div><div class="line">           <span class="keyword">return</span> <span class="keyword">null</span>;</div><div class="line">       &#125;</div><div class="line">       Collections.sort(sizeList, (lhs, rhs) -&gt; &#123;</div><div class="line">           <span class="keyword">if</span> (lhs.width * lhs.height &lt; rhs.width * rhs.height) &#123;</div><div class="line">               <span class="keyword">return</span> <span class="number">1</span>;</div><div class="line">           &#125; <span class="keyword">else</span> <span class="keyword">if</span> (lhs.width * lhs.height &gt; rhs.width * rhs.height) &#123;</div><div class="line">               <span class="keyword">return</span> -<span class="number">1</span>;</div><div class="line">           &#125; <span class="keyword">else</span> &#123;</div><div class="line">               <span class="keyword">return</span> <span class="number">0</span>;</div><div class="line">           &#125;</div><div class="line">       &#125;);</div><div class="line"></div><div class="line">       <span class="comment">// 对于很多机型，符合4：3摄像头比较多，而且与预览窗口大小比较适合</span></div><div class="line">       <span class="keyword">double</span> surfaceRate = <span class="number">0.75</span>;</div><div class="line"></div><div class="line">       <span class="keyword">for</span> (Camera.Size size : sizeList) &#123;</div><div class="line">           <span class="keyword">double</span> previewRate = (<span class="keyword">double</span>) size.height / (<span class="keyword">double</span>) size.width;</div><div class="line">           <span class="keyword">if</span> (Math.abs(previewRate - surfaceRate) &lt; RATE_BOUNDS) &#123;</div><div class="line">               <span class="keyword">return</span> size;</div><div class="line">           &#125;</div><div class="line">       &#125;</div><div class="line"></div><div class="line">       <span class="keyword">return</span> sizeList.get(<span class="number">0</span>);</div><div class="line">   &#125;</div></pre></td></tr></table></figure>
<p>这两个不同的方法分别获取支持的预览分辨率（Preview）和拍照分辨率（Picture），而且这两个数组可能数据不一致，因此在设置时不能保证设置相同的分辨率，只能使用相近的宽高比。比如在本需求中，首先将分辨率数组由大到小排序，然后选取其中宽高比中第一个接近4：3的（比值差值小于0.1）。目前来讲，一般摄像头都会有比较大的支持16：10、16：9及4：3等主流比例的分辨率。这里选取的比例主要看你开发使用的SurfaceView等容器的比例，否则会有比较强烈的拉伸感。若是没有设置预览&amp;拍照分辨率，那系统会选取默认最小的分辨率，基本就没法看了 ╥﹏╥…</p>
</li>
<li><p>其次，还会有模糊的状态，主要原因是摄像头对焦使用的问题</p>
</li>
</ul>
</li>
</ul>
<figure class="highlight plain"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div></pre></td><td class="code"><pre><div class="line">// 获取支持分辨率</div><div class="line">mCamera.getParameters().getSupportedFocusModes();</div><div class="line">parameters.setFocusMode(Camera.Parameters.FOCUS_MODE_CONTINUOUS_PICTURE);</div></pre></td></tr></table></figure>
<p>与设置分辨率类似，同样摄像头支持的对焦模式不尽相同，因此需要获取到Support的模式，然后根据需求的优先级设置相应的对焦模式。文档可以看<a href="https://developer.android.com/reference/android/hardware/Camera.Parameters.html" target="_blank" rel="external">https://developer.android.com/reference/android/hardware/Camera.Parameters.html</a>，拍照可以使用FOCUS_MODE_CONTINUOUS_PICTURE，录像可以使用FOCUS_MODE_CONTINUOUS_VIDEO。</p>
<h3 id="Camera状态管理"><a href="#Camera状态管理" class="headerlink" title="Camera状态管理"></a>Camera状态管理</h3><p>Camera参数配置完成后，会有各种状态，本次开发主要涉及到以下几种：</p>
<ol>
<li>预览开启状态 mCamera.startPreview(); </li>
<li>预览停止状态 mCamera.stopPreview();</li>
<li>进行拍照 mCamera.takePicture(null, null, mCallback);</li>
<li>闪光灯状态 parameters.setFlashMode(Camera.Parameters.FLASH_MODE_OFF/ON);</li>
</ol>
<ul>
<li><p>首先，如果想让SurfaceView展示摄像头预览的内容，需要调用startPreview才能开始有数据流输入，之后会根据设置的聚焦参数逐渐自动聚焦，对于需要点击按钮进行拍照的，可以通过调用takePicture方法获取回调处理拍照的数据。</p>
</li>
<li><p>这里我们会踩到第一个坑，就是对stopPreview的处理。部分手机调用takePicture方法后，会自动停止预览，保留拍照前的最后一帧画面，而部分手机不会停止预览，你会发现即便点了拍照按钮，预览仍然继续，因此拍照后是否停止预览需要手动调用执行stopPreview，不能够依赖系统的默认效果（许多硬件相关开发都是要遵循这个原则啊~）</p>
</li>
<li><p>接下来，就是与生命周期相关了，无论你的代码是在Activity or Fragment中，当回调到onPause时，都需要手动释放当前持有的摄像头，否则其他页面无法持有手机的摄像头… 而这时候则会遇到第二个坑，当你onResume时，摄像头的状态无法保证。如你开启了闪光灯，onStop时释放摄像头，onResume时重新持有，这时候闪光灯是开是关？答案就是有些手机是开着的… 有些手机是关着的… 而且这个你通过Camera.getParameters方法获取的摄像头状态不一定是准确的！这个你敢信！因此需要我们手动在onSaveInstanceState中保留我们想要的状态数据，并在onRestoreInstanceState重新赋值属性，才能保证我们的逻辑没有错误。</p>
</li>
<li><p>然后点击拍照，你就有可能遇到下一个坑… 也许你还记得上文提到的摄像头安装方向导致预览方向与预期不一致的问题，然后我告诉你，拍照也会有这个问题 ﾍ(;´Д｀ﾍ) 同样的，在你获取拍照数据后，需要手动旋转Bitmap，详细逻辑与上文类似，不再赘述</p>
</li>
</ul>
<h3 id="权限处理"><a href="#权限处理" class="headerlink" title="权限处理"></a>权限处理</h3><p>作为Android开发，大家都知道在Android 6.0以后部分权限需要动态申请，而不是像之前那样安装时就会直接获取，摄像头则在需要申请的那组危险权限中（确实能理解，你可以想想后台一个推送，或一个长链发送个数据，你的摄像头自动打开开始录像…）</p>
<p>默认情况下，如果没有申请权限直接调用Camera相关的方法，会直接崩溃，如调用startPreview。而有些ROM自以为人性化的自动会帮你申请权限，防止App崩溃，但作为开发人员无法依赖这种ROM提供的功能，还是每次要自己申请。而权限主要可以分为3种授权状态，<strong>授权</strong>、<strong>询问</strong>、<strong>拒绝</strong>。其中授权就是该App可以任意调用该权限，询问是每次调用关键方法都要申请，需要用户手点同意才会继续执行回调。拒绝则是连弹窗都不会弹，因此需要把部分代码try catch住，并在其exception部分弹出提示，让用户手动赋予App相应的权限。</p>
<p>这里并不是摄像头开发的重点，而且很多其他需要权限的业务开发也都会涉及到，不详述，但还是说说踩住的坑… 主要的问题就在这个申请权限部分，不同的ROM处理不同，这就是<strong>无底深渊</strong></p>
<ol>
<li>有些ROM权限弹窗不会阻塞主线程，你会发现你的代码仍然继续执行并在无权限的情况下，走到了Exception中，用户啥操作都没干呢，但系统把你的提示文案都弹了</li>
<li>有些ROM权限判断有误…明明手机上应用程序权限设置的是询问 or 拒绝，但代码中获取到的Granted就是true，真是不知道咋搞的 还是信任try catch多一点，不然上线一定崩崩崩 ⊙﹏⊙|||</li>
</ol>
<h2 id="总结"><a href="#总结" class="headerlink" title="总结"></a>总结</h2><p>至此，本文说了许多开发拍照相关功能会踩到的坑。其实本期翻译的业务需求开发还涉及到很多其他内容，如jpg格式图片压缩、方向感应、动画等，但与拍照相关的共性不是很多，以后其他人开发很难会遇到，也就不说了… </p>
<p>还是多开发这种需求，才能体会到所谓的Android碎片化，一个屏幕分辨率适配还是无法体现出我大Android碎片的恶心程度的！    </p>

       

