---

layout: post
title:  "Android 版本插桩技术方案"
date:   2017-04-14 1:05:00
catalog:  true
tags:

   - 版本插桩
    
   
---
# 版本插桩需求
需要提供两个新接口：

1. 提供当前版本“是否为全新安装的”判断方法，供各业务线调用。
2. 如果如果“非全新安装”，输出“被覆盖安装版本的信息”：

>
     1、版本号
     2、渠道号
     3、build号
     4、安装时间
     

**该信息任一字段不同均视为不同版本，字段信息后续版本会根据情况不断补充。**

# 技术实现方案

## 流程图

```flow

st=>start: App冷启动
st1=>operation: 读取当前运行版本信息(VerInfo)
st2=>operation: 读取存储的当前版本信息(CurVerInfo)
con1=>condition: 比对VerInfo与CurVerInfo是否一致
st3=>operation: LastVerInfo=CurVerInfo,
CurVerInfo=VerInfo
end=>end: 结束

st->st1->st2->con1
con1(yes)->end
con1(no)->st3->end
```

说明如下：
封装版本信息实体VerInfo，并存储当前版本信息（CurVerInfo）和被覆盖版本信息（LastVerInfo）.

1. App冷启动，读取运行时版本信息（VerInfo）和存储的当前版本信息（CurVerInfo）；
2. 比对VerInfo和CurVerInfo是否一致（任一字段不同均视为不同版本）；
3. 一致，流程结束；
4. 不一致，更新LastVerInfo和CurVerInfo


### 常见case

| case | 全新安装 | 覆盖信息 |
| :-: | :-: | :-: |
| 808之后相同版本覆盖安装 | N | Y |
| 808.1002覆盖808.1001 | N | Y |
| 808覆盖806 | N | N |


### 测试方法

在我的->设置->关于高德地图->10次点击logo->文本的末尾当前版本信息和被覆盖版本信息

## 技术细节：

1. 判断是否为全新安装


```

public static boolean isFirstInstall() {
    try {
        long firstInstallTime =   App.getContext().getPackageManager().getPackageInfo(getPackageName(), 0).firstInstallTime;
        long lastUpdateTime = App.getContext().getPackageManager().getPackageInfo(getPackageName(), 0).lastUpdateTime;
        return firstInstallTime == lastUpdateTime;
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
        return false;
    }
}



public static boolean isInstallFromUpdate() {
    try {
        long firstInstallTime =   App.getContext().getPackageManager().getPackageInfo(getPackageName(), 0).firstInstallTime;
        long lastUpdateTime = App.getContext().getPackageManager().getPackageInfo(getPackageName(), 0).lastUpdateTime;
        return firstInstallTime != lastUpdateTime;
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
        return false;
    }
}
```

参考：

[http://stackoverflow.com/questions/26352881/detect-if-new-install-or-updated-version-android-app](http://stackoverflow.com/questions/26352881/detect-if-new-install-or-updated-version-android-app)

