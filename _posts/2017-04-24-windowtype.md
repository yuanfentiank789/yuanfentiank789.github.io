---

layout: post
title:  "Android入门之窗口类型"
date:   2017-04-24 1:05:00
catalog:  true
tags:

   - 悬浮窗
   - windowmanager
       
   
---

更详细的了解请参考：[http://blog.csdn.net/ritterliu/article/details/39318859](http://blog.csdn.net/ritterliu/article/details/39318859)

  <div id="article_content" class="article_content">

<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 从WmS的角度看，一个窗口并不是Window类，而是一个View类。WmS收到用户消息后，需要把消息派发到窗口，View类本身并不能直接接收WmS传递过来的消息，真正接收用户消息的必须是IWindow类，而实现IWindow类的是ViewRoot.W类，每一个W内部都包含了一个View变量。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;WmS并不介意该窗口（View）是属于哪个应用程序的，WmS会按一定的规则判断哪个窗口处于活动状态，然后把用户消息给W类，W类再把用户消息传递给内部的View变量，剩下的消息处理就由View对象完成。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;Framework定义了三种窗口类型，三种类型的定义在WindowManager的LayoutParams中。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 第一种窗口类型为应用窗口，所谓的应用窗口是指该窗口对应一个Activity，由于加载Activity是由AmS完成的，因此，对于应用程序来说，要创建一个应用类窗口，只能在Activity内部完成。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 应用窗口包含以下几类：</span></p>
<p>
</p><table border="1" width="900" cellspacing="1" cellpadding="1">
<tbody>
<tr>
<td><span style="font-size:14px">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 定义</strong></span></td>
<td><span style="font-size:14px"><strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 意义</strong></span></td>
</tr>
<tr>
<td><span style="font-size:14px">FIRST_APPLICATION_WINDOW = 1</span></td>
<td><span style="font-size:14px">第一个普通应用窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_BASE_APPLICATION = 1</span></td>
<td><span style="font-size:14px">基窗口，所有其他类型的应用窗口将出现在基窗口上层</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION = 2</span></td>
<td><span style="font-size:14px">普通应用窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_STARTING = 3</span></td>
<td><span style="font-size:14px">应用程序启动时先显示此窗口，当真正的窗口配置完成后，此窗口被关闭</span></td>
</tr>
<tr>
<td><span style="font-size:14px">LAST_APPLICATION_WINDOW = 99</span></td>
<td><span style="font-size:14px">最后一个应用窗口</span></td>
</tr>
</tbody>
</table>
<br>
<p></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 所有Activity默认的窗口类型都是TYPE_APPLICATION，WmS在进行窗口叠加时，会动态改变应用窗口的层值，但层值不会大于99。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 第二种窗口类型是子窗口，子窗口是指该窗口必须要有一个父窗口，父窗口可以是一个应用类型窗口，也可以是任何其他类型的窗口。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 子窗口包含以下几类：</span></p>
<p><span style="font-size:18px"></span>
</p><table border="1" width="900" cellspacing="1" cellpadding="1">
<tbody>
<tr>
<td><span style="font-size:14px"><strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 定义</strong></span></td>
<td><span style="font-size:14px"><strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 意义</strong></span></td>
</tr>
<tr>
<td><span style="font-size:14px">FIRST_SUB_WINDOW = 1000</span></td>
<td><span style="font-size:14px">第一个子窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_PANEL = FIRST_SUB_WINDOW</span></td>
<td><span style="font-size:14px">应用窗口的子窗口，PopupWindow的默认类型</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_MEDIA = FIRST_SUB_WINDOW + 1</span></td>
<td><span style="font-size:14px">用来显示Media的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_SUB_PANEL = FIRST_SUB_WINDOW + 2</span></td>
<td><span style="font-size:14px">TYPE_APPLICATION_PANEL的子窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_ATTACHED_DIALOG = FIRST_SUB_WINDOW + 3</span></td>
<td><span style="font-size:14px">OptionMenu、ContextMenu的默认类型</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_APPLICATION_MEDIA_OVERLAY = FIRST_SUB_WINDOW + 4</span></td>
<td><span style="font-size:14px">TYPE_APPLICATION_MEDIA的重影窗口，显示在TYPE_APPLICATION_MEDIA和应用窗口之间</span></td>
</tr>
<tr>
<td><span style="font-size:14px">LAST_SUB_WINDOW = 1999</span></td>
<td><span style="font-size:14px">最后一个子窗口</span></td>
</tr>
</tbody>
</table>
<br>
&nbsp; &nbsp; &nbsp; &nbsp; 创建子窗口时，客户端可以指定窗口类型介于1000-1999之间，而WmS在进行窗口叠加时，会动态调整层值。<p></p>
<p><span style="font-size:18px"><br>
</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 第三种窗口类型是系统窗口，系统窗口不需要对应任何Activity，也不需要有父窗口，对于应用程序而言，理论上是无法创建系统窗口的，因为所有的应用程序都没有这个权限，然而系统进程却可以创建系统窗口。</span></p>
<p><span style="font-size:18px">&nbsp; &nbsp; &nbsp; &nbsp; 系统窗口有以下类型：</span></p>
<p><span style="font-size:18px"></span>
</p><table border="1" width="900" cellspacing="1" cellpadding="1">
<tbody>
<tr>
<td><span style="font-size:14px"><strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 定义</strong></span></td>
<td><span style="font-size:14px"><strong>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 意义</strong></span></td>
</tr>
<tr>
<td><span style="font-size:14px">FIRST_SYSTEM_WINDOW = 2000</span></td>
<td><span style="font-size:14px">第一个系统窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_STATUS_BAR = FIRST_SYSTEM_WINDOW</span></td>
<td><span style="font-size:14px">状态栏窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SEARCH_BAR = FIRST_SYSTEM_WINDOW +1</span></td>
<td><span style="font-size:14px">搜索条窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_PHONE = FIRST_SYSTEM_WINDOW + 2</span></td>
<td><span style="font-size:14px">来电显示窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SYSTEM_ALERT = FIRST_SYSTEM_WINDOW + 3</span></td>
<td><span style="font-size:14px">警告对话框</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_KEYGUARD = FIRST_SYSTEM_WINDOW + 4</span></td>
<td><span style="font-size:14px">屏保</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_TOAST = FIRST_SYSTEM_WINDOW + 5</span></td>
<td><span style="font-size:14px">Toast对应的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SYSTEM_OVERLAY = FIRST_SYSTEM_WINDOW + 6</span></td>
<td><span style="font-size:14px">系统覆盖窗口，需要显示在所有窗口之上</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_PRIORITY_PHONE = FIRST_SYSTEM_WINDOW + 7</span></td>
<td><span style="font-size:14px">在屏幕保护下的来电显示窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SYSTEM_DIALOG = FIRST_SYSTEM_WINDOW + 8</span></td>
<td><span style="font-size:14px">滑动状态条后出现的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_KEYGUARD_DIALOG = FIRST_SYSTEM_WINDOW + 9</span></td>
<td><span style="font-size:14px">屏保弹出的对话框</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SYSTEM_ERROR = FIRST_SYSTEM_WINDOW + 10</span></td>
<td><span style="font-size:14px">系统错误窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_INPUT_METHOD = FIRST_SYSTEM_WINDOW + 11</span></td>
<td><span style="font-size:14px">输入法窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_INPUT_METHOD_DIALOG = FIRST_SYSTEM_WINDOW + 12</span></td>
<td><span style="font-size:14px">输入法中备选框对应的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_WALLPAPER = FIRST_SYSTEM_WINDOW + 13</span></td>
<td><span style="font-size:14px">墙纸对应的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_STATUS_BAR_PANEL = FIRST_SYSTEM_WINDOW + 14</span></td>
<td><span style="font-size:14px">滑动状态条后出现的窗口</span></td>
</tr>
<tr>
<td><span style="font-size:14px">TYPE_SECURE_SYSTEM_OVERLAY = FIRST_SYSTEM_WINDOW + 15</span></td>
<td><span style="font-size:14px">安全系统覆盖窗口，显示在所有窗口之上。</span></td>
</tr>
<tr>
<td><span style="font-size:14px">LAST_SYSTEM_WINDOW = 2999</span></td>
<td><span style="font-size:14px">最后一个系统窗口</span></td>
</tr>
</tbody>
</table>
<br>
<p></p>
<p><span style="font-size:18px"><br>
<br>
</span></p>
   
</div>