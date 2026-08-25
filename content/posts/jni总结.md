---
title: "JNI 总结"
date: 2026-08-14T00:00:00+08:00
lastmod: 2026-08-14T00:00:00+08:00
description: "Android Studio 下使用 JNI 调用 C/C++ 本地代码的流程总结"
tags: ["Android", "JNI"]
categories: ["技术"]
---

# JNI
JNI是指Java Native Interface，在编写Java程序时需要使用本地代码（C/C++）可以使用这一技术。
通常来说，要使用的本地代码有两种：C/C++源代码和经过编译的库（用在Android应用上的是.so库，需要根据运行的设备编译对应版本）。
使用JNI要做的几个部分有：确定接口，编写Java和cpp文件，编写cmake文件，加载库，使用函数。这几个步骤理论上没有顺序，按照我的实践结果，有的步骤按逻辑来执行比较方便。

## 前期配置
我的开发IDE是Android Studio，需要在项目中先配置好NDK和cmake插件。
在开发时，我选择了新建一个模块作为JNI部分，虽然多了一层的封装，但是自己看起来比较清楚。这里有一个小问题，在gradle版本为4.1.1时，会出现新建native Lib模块不能编译的问题，我将gradle版本升级到7.4.2解决了这个问题。如果选择新建moudle，还要在app的`build.gradle`中加入引用
```build.gradle
...
dependencies {  
    ... 
    implementation project(path : ':nativeLib')  
}
```
如果选择在app中直接加入也很简单，在Java同级目录下创建cpp目录，并在cpp文件夹中创建CMakeLists.txt等文件就可以。
当然，以上两种都可以在创建app或模块时就选择native lab，IED会自动创建相关目录和文件。


## 确定接口
这里的接口指的是在Java中会调用的接口。在Java文件中，要加入`native`。
```java
public static native int init();
```
这里的接口也对应要编写的cpp文件。

## Java文件加载库
在Java代码中除了要处理接口，还要引入库，我们编写的cpp文件也会编译成库，不要忘记加上这个库。
```java
	// cpp文件的库
	System.loadLibrary("nativeLib");  
	// 其他库，对应名称libexample.lib
	System.loadLibrary("example");
```
## 编写cpp文件
### 生成接口定义
Java中确定的native接口也需要在cpp中实现。
Java确定cpp文件中的函数名称有一套规则：`Java_{package_and_classname}_{function_name}(JNI arguments)`。包名中的点换成单下划线。需要说明的是生成函数中的两个参数：  
1. JNIEnv *：这是一个指向JNI运行环境的指针，后面我们会看到，我们通过这个指针访问JNI函数  
2. jobject：这里指代java中的this对象

我不推荐按照这个规则直接去写，有两种方法可以生成Java能找到的cpp函数名：
1. 将javah.exe加入环境变量并使用以下命令
```cmd
javac -h . nativeLab.java
java nativeLib.java
javah -jni nativeLib.java
```
2. 使用Android Studio 外部工具，将javah.exe作为外部工具直接使用生成头文件。
我们可以直接参照生成的头文件来编写cpp文件，在编译中，这个头文件不是必要的。
### 函数内容
java会将一些参数传给cpp，所以这些参数需要我们进行处理的转化。
#### 原始类型
jint, jbyte, jshort, jlong, jfloat, jdouble, jchar, jboolean这些分别对应这java的int, byte, short, long, float, double, char and boolean。
#### string
1. 使用`GetStringUTFChars()`函数来将`jstring`转换成`char *  `
2. 然后进行需要的数据处理  
3. 使用`NewStringUTF()`函数来将`char *`转换成`jstring`，并且返回
#### array
1. 使用`jint* GetIntArrayElements(JNIEnv *env, jintArray a, jboolean *iscopy)将jintarray`转换成C的`jint[]  `
2. 使用`jintArray NewIntArray(JNIEnv *env, jsize len)`函数来分配一个`len`字节大小的空间，然后再使用`void SetIntArrayRegion(JNIEnv *env, jintArray a, jsize start, jsize len, const jint *buf)`函数将`jint[]`中的数据拷贝到`jintArray`中去。
3. 使用`ReleaseIntArrayElements (jintArray array, jint* elems,
jint mode)`释放。其中，jint mode 参数意义：模式 0 : 刷新 Java 数组 , 释放 C/C++ 数组 ；模式 1 ( `JNI_COMMIT` ) : 刷新 Java 数组 , 不释放 C/C ++ 数组 ；模式 2 (` JNI_ABORT` ) : 不刷新 Java 数组 , 释放 C/C++ 数组。
#### object
可以在native代码中构造jobject和jobjectarray，通过调用NewObject() 和 newObjectArray()函数，然后讲它们返回给java代码。
#### 访问java对象
1. 调用`GetObjectClass()`获得目标对象的类引用  
2. 从获得的类引用中获得`Field ID`来访问变量，你需要提供这个变量的名字，变量的描述符（也称为签名）。
> 对于java类而言，描述符是这样的形式：“Lfully-qualified-name;”(注意最后有一个英文半角分号)，其中的包名点号换成斜杠(/)，比如java的Stirng类的描述符就是“Ljava/lang/String;”。对于基本类型而言，I代表int，B代表byte，S代表short，J代表long，F代表float，D代表double，C代表char，Z代表boolean。对于array而言，使用左中括号”[“来表示，比如“[Ljava/lang/Object;”表示Object的array，“[I”表示int型的array。  
3. 基于上面获得的`Field ID`，使用`GetObjectField()` 或者 `Get_primitive-type_Field()`函数来从中解析出我们想要的数据  
4. 使用`SetObjectField() `或者 `Set_primitive-type_Field()`函数来修改变量

### 示例
```c
 #include <jni.h>
#include <string>
#include <android/log.h>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_myapplication_MainActivity_stringFromJNI(
	JNIEnv* env,
	jobject /* this */) {   
	std::string hello = "Hello from C++";
	// 使用 android log输出日志
	__android_log_print(ANDROID_LOG_INFO, "log", "Hello log from JNI function");
	return env->NewStringUTF(hello.c_str());
```
-   `extern "C"` 告诉编译器按照 C 语言的规则处理函数 `stringFromJNI`。
-   `JNIEXPORT` 表示该函数将被导出供 JNI 调用。
-   `JNICALL` 是一个宏，用于设置正确的调用约定。

#### print_log
示例中添加了`#include <android/log.h>` 用于使用Android log 方法：`__android_log_print`
> 这里不可以直接使用C原生的 printf("Hello log from JNI function\n")
因为在 Android 开发中，`printf` 输出的内容通常不会直接显示在 Logcat 中。Android 应用默认会将 `stdout` 和 `stderr` 重定向到 `/dev/null`，因此 `printf` 的输出不会在 Logcat 中出现

## 编写cmake
如果使用IDE建立native部分，就会自动生成CMakeLists.txt文件，主要需要关注的有这几部分
- `target_include_directories`关联到第三方库头文件
- `add_library`加入第三方库
- `set_target_properties`指明第三方库路径
- `target_link_libraries`关联到ndk中
- `find_library`添加ndk原生库

## 调用
在app需要调用的部分引入包直接使用就可以了。
