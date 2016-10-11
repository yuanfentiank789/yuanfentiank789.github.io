---
layout: post
title:  "Android暗码"
date:   2016-10-11 1:05:00
catalog:  true
tags:

   - android
   - 暗码
   - secret code
     


---

## 什么是暗码？

在android系统中，暗码就是类似这种样式的字符串： *#*#<code>#*#*

如果这样的系统暗码执行，系统会触发下面的方法：（来自 AOSP Android Open Source Project）

    static private boolean handleSecretCode(Context context, String input) {
    int len = input.length();
    if (len > 8 && input.startsWith("*#*#") && input.endsWith("#*#*")) {
        Intent intent = new Intent(TelephonyIntents.SECRET_CODE_ACTION,
                Uri.parse("android_secret_code://" + input.substring(4, len - 4)));
        context.sendBroadcast(intent);
        return true;
    }

    return false;
    }
    
## 如何运行暗码?

有两种方式可以执行运行暗码：

直接在手机电话拨号界面输入暗码，例如：*#*#123456789#*#*



     String secretCode = "123456789";
     Intent intent = new Intent(Intent.ACTION_DIAL);    
     intent.setData(Uri.parse("tel:*#*#" + secretCode + "#*#*"));
     startActivity(intent);
     
或者直接在代码中进行调用：     
 
    String secretCode = "123456789";
    String action = "android.provider.Telephony.SECRET_CODE";
    Uri uri = Uri.parse("android_secret_code://" + secretCode);
    Intent intent = new Intent(action, uri);
    sendBroadcast(intent);
    
## 如何创建自定义暗码？

在你的应用AndroidManifest.xml文件中，增加以下代码，用来定义手机暗码。

不管什么时候暗码 *#*#123456789#*#* 被触发，你都将收到对应的广播。

    <receiver android:name=".MySecretCodeReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SECRET_CODE" />
        <data android:scheme="android_secret_code" android:host="123456789" />
    </intent-filter>
    </receiver>

