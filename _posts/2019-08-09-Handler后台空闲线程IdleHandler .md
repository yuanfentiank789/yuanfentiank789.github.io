---
layout: post
title:  Handler后台空闲线程IdleHandler
date:   2019-08-09 1:05:00
catalog:  true
tags:
    - IdleHandler 
    
         
       

---


在Android中，我们可以处理Message，这个Message我们可以立即执行也可以delay 一定时间执行。Handler线程在执行完所有的Message消息，它会wait，进行阻塞，知道有心的Message到达。如果这样子，那么这个线程也太浪费了。MessageQueue提供了另一类消息，IdleHandler。
 
 
示例代码如下：

```
package com.example.testhandler;
 
import android.os.Bundle;
import android.os.Handler;
import android.os.HandlerThread;
import android.os.Looper;
import android.os.Message;
import android.os.MessageQueue;
import android.app.Activity;
import android.util.Log;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
 
public class MainActivity extends Activity {
	
	private Handler mHandler;
	
	private int mWhat = 0;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		mHandler = new Handler(){
 
			@Override
			public void handleMessage(Message msg) {
				Log.d("hlwang", "我在执行，你想回来，我用平底锅打飞你！");
				try {
					Thread.sleep(1500);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				super.handleMessage(msg);
			}
			
		};
		
		Button btn = (Button) findViewById(R.id.btn);
		btn.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				for(int i=0;i<10;i++){
					mHandler.sendEmptyMessage(mWhat);
				}
			}
		});
	}
 
	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		getMenuInflater().inflate(R.menu.main, menu);
		return true;
	}
 
	@Override
	protected void onResume() {
		super.onResume();
		
		Looper.myQueue().addIdleHandler(new MyIdleOnce());
		Looper.myQueue().addIdleHandler(new MyIdleKeep());
		
	}
	
	class MyIdleKeep implements MessageQueue.IdleHandler{
		/**
		 *返回值为true，则保持此Idle一直在Handler中，否则，执行一次后就从Handler线程中remove掉。
		 */
		@Override
		public boolean queueIdle() {
			Log.d("hlwang","我是空闲线程,我还会回来的！");
			return true;
		}
		
	}
 
	class MyIdleOnce implements MessageQueue.IdleHandler{
 
		@Override
		public boolean queueIdle() {
			Log.d("hlwang","我是初恋，我只在你的生命中出现一次，我发誓，你会想我的！");
			return false;
		}
		
	}
}

```

此Activity有两个IdleHandler消息，我们执行此Activity时，MyIdleKeep消息和MyIdleOnce会依次执行。如果IdleHandler的queueIdle方法返回false，那么IdleHandler执行完，就会从IdleHandler移除。

```
03-10 10:11:29.556: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:29.556: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:29.556: D/hlwang(21985): 我是初恋，我只在你的生命中出现一次，我发誓，你会想我的！
03-10 10:11:29.556: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:29.576: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:29.576: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:29.576: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:30.396: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:30.396: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:30.396: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:32.276: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:32.276: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:32.276: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:32.326: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:34.386: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:36.396: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:38.386: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:40.396: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:42.386: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:44.396: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:46.436: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:48.436: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:50.446: D/hlwang(21985): 我在执行，你想回来，我用平底锅打飞你！
03-10 10:11:52.696: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:52.696: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:52.696: D/hlwang(21985): 我是空闲线程,我还会回来的！
03-10 10:11:52.796: D/hlwang(21985): 我是空闲线程,我还会回来的！

```

另外，开源库LeakCanary的AndroidWatchExecutor中也用了idlehandler来触发内存泄漏的检测。

