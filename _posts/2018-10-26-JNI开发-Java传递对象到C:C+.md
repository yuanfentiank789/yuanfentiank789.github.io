---
layout: post
title:  JNI开发-Java传递对象到C/C+
date:   2018-10-26 1:05:00
catalog:  true
tags:
    - C++
    - JNI
         
       

---

JNI开发有时需要Java把对象作为参数传递到C/C++，此篇主要讲解Java传递Student对象到C/C++;

1 定义com.niubashaoye.simple.jni.StuInfo类;

```

public class StuInfo {
	private int stuId;
	private String stuName;
	private int stuAge;
	private String className;
 
	public StuInfo(int stuId, String stuName, int stuAge, String className) {
		super();
		this.stuId = stuId;
		this.stuName = stuName;
		this.stuAge = stuAge;
		this.className = className;
	}
    
          //getter()
          //setter()
 
	@Override
	public String toString() {
		return "StuInfo [stuId=" + stuId + ", stuName=" + stuName + ", stuAge=" + stuAge + ", className=" + className
				+ "]";
	}
 
}

```

2 添加native函数

```
public class JNITools {
	static {
		System.loadLibrary("TestDemo");
	 }	
	/**
	 * 设置学生信息
	 * 
	 * @param stuInfo
	 * @return
	 */
	public native void setStuInfo(StuInfo stuInfo);

}
```

3 C/C++文件

3.1 添加StuInfo结构体

```
typedef struct {
	int stuId;
	char stuName[50];
	int stuAge;
	char className[50];
} StuInfo;
```

3.2 获取StuInfo对象

```

JNIEXPORT void JNICALL Java_com_niubashaoye_simple_jni_JNITools_setStuInfo(
		JNIEnv *env, jobject obj, jobject jobj) {
	StuInfo stuInfo;
	LOGI("====setStu===1====");
	//获取jclass的实例
	jclass jcs = env->FindClass("com/niubashaoye/simple/jni/StuInfo");
	
	//获取StuInfo的字段ID
	jfieldID fileId = env->GetFieldID(jcs, "stuId", "I");
	jfieldID nameId = env->GetFieldID(jcs, "stuName", "Ljava/lang/String;");
	jfieldID ageId = env->GetFieldID(jcs, "stuAge", "I");
	jfieldID classId = env->GetFieldID(jcs, "className", "Ljava/lang/String;");
	
	//把字段Id设置到结构体中
	stuInfo.stuId = env->GetIntField(jobj, fileId);
	jstring nameStr = (jstring) env->GetObjectField(jobj, nameId);
	const char *locstr = env->GetStringUTFChars(nameStr, 0);
	strcpy(stuInfo.stuName, locstr);
	stuInfo.stuAge = getIntByObjInfo(env, jcs, jobj, "stuAge");
	jstring classStr = (jstring) env->GetObjectField(jobj, classId);
	const char *cstr = env->GetStringUTFChars(classStr, 0);
	strcpy(stuInfo.className, cstr);
	LOGI("====setStu===4====");
	LOGI("传递到C的StuInfo：stuId：%d;stuName：%s;stuAge：%d;className：%s",
			stuInfo.stuId, stuInfo.stuName, stuInfo.stuAge, stuInfo.className);
}

```
4 Java调用native函数

```
StuInfo stuInfo = new StuInfo(1000, "牛八少爷", 22, "高三一班");
JNITools jniTools= new JNITools(); 
jniTools.setStuInfo(stuInfo);
```


