---

layout: post
title:  "使用MAT比较多个heap dump文件"
date:   2017-12-26 1:06:00
catalog:  true
tags:

   - MAT
   - heap比较

---
   
   
<div id="article_content" class="article_content csdn-tracking-statistics tracking-click" data-mod="popu_519" data-dsm="post">
                        <p><a target="_blank"></a>&nbsp;</p><h1 align="center">使用MAT比较多个heap dump文件</h1><p>&nbsp;</p><p>调试内存泄露时，有时候适时比较2个或多个heap dump文件是很有用的。这时需要生成多个单独的HPROF文件。</p><p>下面是一些关于如何在MAT里比较多个heap dumps的内容（有一点复杂）：</p><p>1.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一个HPROF 文件(usingFile &gt; Open Heap Dump ).</p><p>2.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开Histogram view.</p><p align="center"><img src="http://img.blog.csdn.net/20140812114050379?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc29kaW5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt=""></p><p align="center">图1. Histogram View按钮</p><p>3.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在NavigationHistory view里 (如果看不到就从Window &gt; Navigation History找 ), 右击histogram然后选择Add to Compare Basket .</p><p>4.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开第二个HPROF 文件然后重做步骤2和3.</p><p>5.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;切换到Compare Basket view, 然后点击Compare the Results (视图右上角的红色"!"图标)。</p><p>&nbsp;</p><p align="center"><img src="http://img.blog.csdn.net/20140812114108522?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc29kaW5v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt=""></p><p align="center">图2. 对比分析结果</p><p>&nbsp;</p><p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 如上，结果图中，Objects #1所代表的weak.create.hprof比Objects#0所代表的main.hporf多出了一个WeakReferencesActivity；Objects #2更是多出10000个WFObject对象出来，结果很明显。</p>           </div>   
   
       
   

