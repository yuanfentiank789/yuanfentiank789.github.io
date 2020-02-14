---
layout: post
title:  CSS position 属性总结
date:   2019-07-17 1:05:00
catalog:  true
tags:
    - CSS
    - position
           
       

---


<div data-note-content="" class="show-content">
          <div class="show-content-free">
            <p>CSS的position总是属性很容易让人弄混~</p>
<p>为了仔细区别它们，所以今天总结一下CSS的position属性~</p>
<p>下面是总结内容~</p>
<p>有疏漏、错误之处敬请指出！<sub>o(^▽^)o</sub></p>
<hr>
<h2>一、简介</h2>
<p><strong>定义</strong>：position属性规定元素的定位类型。</p>
<p><strong>说明</strong>：这个属性定义建立元素布局所用的定位机制。任何元素都可以定位，不过绝对定位或固定元素会生成一个块级框，而不论该元素本身是什么类型。相对定位元素会相对于它在正常流中的默认位置偏移。</p>
<p><strong>默认值</strong>：static</p>
<p><strong>继承性</strong>：no</p>
<p><strong>形式语法</strong>：static | relative | absolute | sticky | fixed</p>
<p><strong>JavaScript语法</strong>：object.style.position = "absolute"</p>
<p><strong>浏览器支持</strong>：所有主流浏览器都支持position属性。但：任何版本的Internet Explorer(包括IE8)都不支持属性值“inhert”。</p>
<h2>二、取值</h2>
<table>
<thead>
<tr>
<th>值</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td>static</td>
<td>默认值，<code>没有定位</code>。元素出现在正常的流中。（忽略top、right、bottom、left和z-index属性）</td>
</tr>
<tr>
<td>relative</td>
<td>生成<code>相对定位</code>的元素，相对于其正常位置进行定位。</td>
</tr>
<tr>
<td>absolute</td>
<td>生成<code>绝对定位</code>的元素，相对于最近的非static父元素进行定位。绝对定位的元素可以设置外边距（margin），且不会与其他他边距合并。</td>
</tr>
<tr>
<td>fixed</td>
<td>生成<code>绝对定位</code>的元素，相对于浏览器窗口(viewport)进行定位。</td>
</tr>
<tr>
<td>sticky</td>
<td>盒位置根据正常流计算，然后相当于该元素在流中的flow root(BFC)和containing block（最近的块级元素）定位。</td>
</tr>
</tbody>
</table>
<p><strong>注：</strong></p>
<p>文档流：将窗体<code>自上而下</code>分成一行行，并在每行中按<code>从左至右</code>的顺序排放元素，即为文档流。</p>
<p>元素脱离文档流的情况：浮动、绝对定位（absolute、fixed）。</p>
<h2>三、 详解</h2>
<p>我们以一个初始未定位示例为参照，其他属性值将和它进行比较。</p>
<p>[定位参照示例]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/3gwbeu3v/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/3gwbeu3v/</a>）<br>
</p><div class="image-package">
<div class="image-container" style="max-width: 570px; max-height: 468px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 82.11%;"></div>
<div class="image-view" data-width="570" data-height="468"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-ea40706683670dce.png" data-original-width="570" data-original-height="468" data-original-format="image/png" data-original-filesize="14747" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-ea40706683670dce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/570/format/webp"></div>
</div>
<div class="image-caption">position-0.png</div>
</div><p></p>
<h3>1. static</h3>
<p>默认值。没有定位，元素出现在正常流中（忽略top，bottom，left，right或者z-index声明）</p>
<p><strong>解释</strong>：position设置为static或不设定position属性时，元素遵循正常的文档流，对象占用文档空间，该定位方式下，top、right、left、bottom、z-index属性是无效的。</p>
<p>示例：[static定位]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/zj2ogz8m/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/zj2ogz8m/</a>）</p>
<p>通过上例，可以看出position属性设置为static和不设置其实是一样的~</p>
<p>所以，通常此属性值可以不设置，除非要<code>覆盖之前的定义。</code>下面举个栗子：</p>
<pre class="hljs css"><code class="css"><span class="hljs-selector-tag">A</span>页面
<span class="hljs-selector-id">#div-1</span>{
  <span class="hljs-attribute">position</span>:absolute；
}
<span class="hljs-selector-tag">B</span>页面
<span class="hljs-selector-id">#div-1</span>{
  <span class="hljs-attribute">position</span>:absolute；
}

</code></pre>
<p>页面B中`position:static;是为了覆盖页面A中position的定义。</p>
<h3>2. relative</h3>
<p>生成相对定位的元素，相对于其正常位置进行定位。</p>
<p><strong>解释：</strong></p>
<p>(1）position设置为relative时，top、right、left、bottom等属性有效，相对其<code>正常位置</code>移。</p>
<p>(2）position设置为relative时，元素遵照正常的文档流，占据文档空间，但是占据的文档空间<code>不会</code>随top、right、left、bottom的偏移而发生变动，也就是说，它后面的元素是依据前一个元素正常位置（即未设置top、right、left、bottom属性之前）进行的定位。</p>
<p>(3）position设置为relative时，如果没有进行任何的top、right、left、bottom设置，元素不会进行任何位置的改变。</p>
<p>示例：[relative定位-1]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/ae8s2cm4/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/ae8s2cm4/</a>）<br>
</p><div class="image-package">
<div class="image-container" style="max-width: 594px; max-height: 480px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 80.81%;"></div>
<div class="image-view" data-width="594" data-height="480"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-674c255bc5e5ebd0.png" data-original-width="594" data-original-height="480" data-original-format="image/png" data-original-filesize="21188" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-674c255bc5e5ebd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/594/format/webp"></div>
</div>
<div class="image-caption">position-1.png</div>
</div><p></p>
<blockquote>
<p>虚线是初始的位置空间。</p>
</blockquote>
<p>由图我们可以看出，相对定位是相对元素原本在文档流中的位置而进行的偏移。并且它后面的元素——second是依据虚线位置，也就是元素原本在文档流中的位置而进行的定位。</p>
<p>好了，我们现在知道了top、right、left、bottom等属性不会改变relative定位的元素所占据的文档空间。，那么margin、padding会改变该元素占据的文档空间吗？我们来试一下：</p>
<p>css代码中添加margin属性：<br>
示例：[relative定位-2]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/vxc9ctqs/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/vxc9ctqs/</a>）<br>
</p><div class="image-package">
<div class="image-container" style="max-width: 664px; max-height: 548px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 82.53%;"></div>
<div class="image-view" data-width="664" data-height="548"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-0cb61867fe569267.png" data-original-width="664" data-original-height="548" data-original-format="image/png" data-original-filesize="42524" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-0cb61867fe569267.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/664/format/webp"></div>
</div>
<div class="image-caption">position-2.png</div>
</div><p></p>
<p>由图可以看出，我们将外边距设置为20px，second元素向下偏移40px，所以margin可以改变元素所占文档空间！同理，padding也可改变元素所占文档空间，这里不多做演示。</p>
<h3>3. absolute</h3>
<p>生成绝对定位的元素，相对于非static定位的第一个父元素进行定位。</p>
<p><strong>解释</strong>：position设置为absolute，元素会脱离文档流，整个元素不再占据文档空间，就只能相对<code>非static</code>定位的<code>第一个</code>父元素进行定位<br>
(1）absolute在无父级是非static定位时以&lt;html&gt;标签作为原点定位，而relative和static方式在最外层时是以&lt;body&gt;标签作为原点定位。&lt;html&gt;标签和&lt;body&gt;标签相差9px左右。</p>
<p>(2）position设置为absolute或fixed时，必须指定left、right、top、bottom属性中的<code>至少一个</code>，否则left、right、top、bottom属性会使用它们的默认值auto，这将导致对象遵从正常的HTML布局规则，在前一个对象之后立即被呈递，简单讲就是都变成relative，会占用文档空间，不会脱离文档流。若多设，比如top和bottom一同存在的话，那么只有top生效；left和right一同存在的话，那么只有left生效。</p>
<p>(3）绝对（absolute）定位对象和相对（relative）定位对象在可视区域之外会导致滚动条出现。而固定（fixed）定位对象放置在可视区域之外，滚动条不会出现。</p>
<p>示例：<br>
[absolute定位-1]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/q325w0k6/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/q325w0k6/</a>）<br>
</p><div class="image-package">
<div class="image-container" style="max-width: 700px; max-height: 360px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 21.2%;"></div>
<div class="image-view" data-width="1698" data-height="360"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-7a9f8cbccc52ae34.png" data-original-width="1698" data-original-height="360" data-original-format="image/png" data-original-filesize="33631" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-7a9f8cbccc52ae34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp"></div>
</div>
<div class="image-caption">position-3.png</div>
</div><p></p>
<p>由此例我们可以发现absolute相对html定位，relative相对body定位。</p>
<p>下面我们再来看看对absolute定位的元素的除static外第一个父元素设置margin/padding，看看会不会对文档空间有影响。</p>
<p>在absolute定位中添加margin/padding属性：</p>
<p>[absolute定位-2]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/59xv1qmx/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/59xv1qmx/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 700px; max-height: 533px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 72.32%;"></div>
<div class="image-view" data-width="737" data-height="533"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-5464c7204bc9d4fd.png" data-original-width="737" data-original-height="533" data-original-format="image/png" data-original-filesize="17977" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-5464c7204bc9d4fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/737/format/webp"></div>
</div>
<div class="image-caption">position-4.png</div>
</div>
<p>由上图我们可以看出，父元素的margin会让子元素的absolute定位跟着偏移，而padding却不会让子元素发生偏移。总结：absolute是根据父元素的<code>border</code>进行的定位。</p>
<h3>4.fixed</h3>
<p>生成绝对定位的元素，相对于浏览器窗口进行定位。</p>
<p><strong>解释：</strong> fixed定位，又称固定定位，它和absolute定位一样，都脱离了文档流，并且能够根据top、right、left、bottom属性进行定位，但不同的是fixed是根据<code>窗口</code>为原点进行偏移定位的，也就是说它不会根据滚动条的滚动而进行偏移。</p>
<p>示例：[fixed定位]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/cubxLga9/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/cubxLga9/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 602px; max-height: 983px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 140.53%;"></div>
<div class="image-view" data-width="602" data-height="846"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-161fafaabc2c3061.png" data-original-width="602" data-original-height="846" data-original-format="image/png" data-original-filesize="17164" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-161fafaabc2c3061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/602/format/webp"></div>
</div>
<div class="image-caption">position-5.png</div>
</div>
<h3>5. sticky</h3>
<p>粘性定位。CSS3新属性。它的表现类似<code>position:relative</code>和<code>position:fixed的</code>合体，在目标区域在屏幕中可见时，它的行为就像<code>position:relative</code>; 而当页面滚动超出目标区域时，它的表现就像<code>position:fixed</code>，它会固定在目标位置。</p>
<p>目前<code>position: sticky;</code>的浏览器兼容性还比较差。</p>
<p>示例：[sticky定位]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/rztoc45w/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/rztoc45w/</a>）</p>
<h3>6. inherit</h3>
<p>规定应该从父元素继承position属性的值。</p>
<p>示例：[inherit定位]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/4uLp2ybv/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/4uLp2ybv/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 536px; max-height: 1102px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 157.46%;"></div>
<div class="image-view" data-width="536" data-height="844"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-8e19aabb419f98ea.png" data-original-width="536" data-original-height="844" data-original-format="image/png" data-original-filesize="19629" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-8e19aabb419f98ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/536/format/webp"></div>
</div>
<div class="image-caption">position-6.png</div>
</div>
<p>运行，我们发现second继承了first的position属性：fixed。同时超出可视区域之外时不会出现滚动条。</p>
<h2>总结</h2>
<h3>1. 相对定位的属性：</h3>
<p>(1）如果设定了top、right、left、bottom等属性，并且父元素<code>没有</code>设定position属性，元素以其父元素的左上角为原点进行定位。</p>
<p>[示例]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/z2L92kqz/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/z2L92kqz/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 514px; max-height: 494px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 96.11%;"></div>
<div class="image-view" data-width="514" data-height="494"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-62a872f16fb5bf41.png" data-original-width="514" data-original-height="494" data-original-format="image/png" data-original-filesize="11607" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-62a872f16fb5bf41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/514/format/webp"></div>
</div>
<div class="image-caption">position-7.png</div>
</div>
<p>(2）如果设定了top、right、left、bottom等属性，并且父元素设定position属性（无论是absolute还是relative），则以父元素的左上角为原点进行定位，位置由top、right、left、bottom决定，但是如果父元素存在padding属性，则以<code>content</code>的左上角为原点进行定位。</p>
<p>[示例]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/2jgk2nzv/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/2jgk2nzv/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 552px; max-height: 542px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 98.19%;"></div>
<div class="image-view" data-width="552" data-height="542"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-27b6e806076a5bbc.png" data-original-width="552" data-original-height="542" data-original-format="image/png" data-original-filesize="12498" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-27b6e806076a5bbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/552/format/webp"></div>
</div>
<div class="image-caption">position-8.png</div>
</div>
<div class="image-package">
<div class="image-container" style="max-width: 700px; max-height: 616px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 85.56%;"></div>
<div class="image-view" data-width="720" data-height="616"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-1b36609a769ac706.png" data-original-width="720" data-original-height="616" data-original-format="image/png" data-original-filesize="29249" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-1b36609a769ac706.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/720/format/webp"></div>
</div>
<div class="image-caption">position-9.png</div>
</div>
<blockquote>
<p>虚线框为正常原始位置。<br>
点线框为content内区域</p>
</blockquote>
<p>我们可以看出，元素是以content为原点进行定位的~</p>
<p>以上两点总结：相对定位总是以父元素左上角为原点进行定位的，如果父元素不存在或没有position属性或position属性值为static，则以浏览器左上角进行定位。</p>
<h3>2. 绝对定位的属性</h3>
<p>(1）如果设定了top、right、left、bottom等属性，并且父元素<code>没有</code>设定position属性，元素以浏览器左上角为原点进行定位，位置由top、right、left、bottom决定。</p>
<p>[示例]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/dw2wfh6n/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/dw2wfh6n/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 552px; max-height: 542px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 98.19%;"></div>
<div class="image-view" data-width="552" data-height="542"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-21d41b86e04946b5.png" data-original-width="552" data-original-height="542" data-original-format="image/png" data-original-filesize="12498" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-21d41b86e04946b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/552/format/webp"></div>
</div>
<div class="image-caption">position-10.png</div>
</div>
<p>由图可以看出，是相对于浏览器左上角进行定位的~</p>
<p>(2）如果设定了top、right、left、bottom等属性，并且父元素设定position属性（无论是absolute还是relative），则以父元素的左上角为原点进行定位，位置由top、right、left、bottom决定。但是父元素存不存在padding属性，对定位原点没有影响。</p>
<p>[示例]（<a href="https://link.jianshu.com?t=https://jsfiddle.net/hysunny/2yL8Lo3k/" target="_blank" rel="nofollow">https://jsfiddle.net/hysunny/2yL8Lo3k/</a>）</p>
<div class="image-package">
<div class="image-container" style="max-width: 700px; max-height: 632px; background-color: transparent;">
<div class="image-container-fill" style="padding-bottom: 90.03%;"></div>
<div class="image-view" data-width="702" data-height="632"><img data-original-src="//upload-images.jianshu.io/upload_images/3779867-7650a128c2f24227.png" data-original-width="702" data-original-height="632" data-original-format="image/png" data-original-filesize="26651" style="cursor: zoom-in;" class="" src="//upload-images.jianshu.io/upload_images/3779867-7650a128c2f24227.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/702/format/webp"></div>
</div>
<div class="image-caption">position-11.png</div>
</div>
<blockquote>
<p>虚线框为正常原始位置。<br>
点线框为border内区域</p>
</blockquote>
<p>我们可以看出，父元素的padding并没有影响到子元素的定位。</p>
<p>由以上两点可以总结出：<br>
若想把一个定位为absolute的元素定位于其父元素内</p>
<p>必须满足两个条件：</p>
<p>1）至少设定top、right、left、bottom中的一个</p>
<p>2）父元素设定position属性（值非static）</p>
<p>完。</p>
<hr>
<p>总结内容参考以下：</p>
<p><a href="https://link.jianshu.com?t=https://developer.mozilla.org/zh-CN/docs/Web/CSS/position" target="_blank" rel="nofollow">MDN: position</a><br>
<a href="https://link.jianshu.com?t=http://www.jb51.net/web/77495.html" target="_blank" rel="nofollow">css中position属性(absolute|relative|static|fixed)概述及应用</a><br>
<a href="https://link.jianshu.com?t=https://segmentfault.com/a/1190000000680773" target="_blank" rel="nofollow">详解css相对定位和绝对定位</a></p>
<p>十分感谢你们的分享<sub>n(*≧▽≦*)n</sub></p>
<p>注：原文章首发于：<a href="https://link.jianshu.com?t=http://www.qdfuns.com/notes/15972/8789b96cfceeeb31786f83fcd68d6ff0.html" target="_blank" rel="nofollow">CSS position属性总结</a>，现迁移至此。</p>

          </div>
        </div>

