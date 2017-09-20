---

layout: post
title:  "Mac OS X下Maven的安装与配置"
date:   2017-09-19 1:05:00
catalog:  true
tags:

   - maven
   
   
   
       
   
---

<div data-note-content="" class="show-content">
          <p>Mac OS X 安装Maven:</p>
<ul>
<li>
<p>下载 <a href="https://maven.apache.org/download.cgi" target="_blank">Maven</a>, 并解压到某个目录。例如/Users/robbie/apache-maven-3.3.3</p>
</li>
<li>
<p>打开<code>Terminal</code>,输入以下命令，设置<code>Maven classpath</code></p>
<pre class="hljs undefined"><code> $ vi ~/.bash_profile</code></pre>
<p>添加下列两行代码，之后保存并退出<code>Vi</code>：</p>
<pre class="hljs undefined"><code> export M2_HOME=/Users/robbie/apache-maven-3.3.3
 export PATH=$PATH:$M2_HOME/bin</code></pre>
</li>
</ul>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/420187-6947fe0af3933d3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/420187-6947fe0af3933d3d.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">bash_profile</div>
</div>
<ul>
<li>
<p>输入命令以使<code>bash_profile</code>生效</p>
<pre class="hljs undefined"><code> $ source ~/.bash_profile</code></pre>
</li>
<li>
<p>输入<code>mvn -v</code>查看<code>Maven</code>是否安装成功</p>
</li>
<li>
<p>如果遇到以下异常，重新编辑<code>bash_profile</code>文件，增加<code>export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_11.jdk/Contents/Home</code>后，并重新运行<code>$ source ~/.bash_profile</code>即可。</p>
</li>
</ul>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/420187-21014c26d4442c05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/420187-21014c26d4442c05.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">exception</div>
</div>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/420187-0b45f0a40e8da2f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/420187-0b45f0a40e8da2f5.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">new bash_profile</div>
</div>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/420187-fee2d6db07d70ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/420187-fee2d6db07d70ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">maven version</div>
</div>



