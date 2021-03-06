##参考
https://blog.n0texpecterr0r.cn/2019/08/09/%e8%b7%9f%e6%88%91%e4%b8%80%e8%b5%b7%e7%94%a8-asm-%e5%ae%9e%e7%8e%b0%e7%bc%96%e8%af%91%e6%9c%9f%e5%ad%97%e8%8a%82%e7%a0%81%e6%8f%92%e6%a1%a9/
https://www.jianshu.com/p/13d18c631549
https://www.jianshu.com/p/44d3c7d46e5f
https://github.com/N0tExpectErr0r/Elapse/blob/master/build.gradle

## 基本介绍

一个基于 ASM 实现的 Gradle Plugin，可以在编译期对注解所指定的方法进行插桩，在方法调用前后执行指定逻辑。

具体实现思路文章可以见此篇文章： [跟我一起用 ASM 实现编译期字节码插桩](http://blog.n0texpecterr0r.cn/?p=752)，由于只是 Demo，**不建议用于正式项目**。

## 使用方式

### 引入

首先需要将 `elapse` 及 `elapse-asm` 导入到项目中。

在项目的 build.gradle 中将其添加到 classpath 中：

```groovy
repositories {
    //...
    maven {
        url uri("repo")
    }
}
 
dependencies {
    // ...
    classpath 'com.n0texpecterr0r.build:elapse-asm:1.0.0'
}
```

在 App module 中将插件 apply：

```groovy
apply plugin: 'com.n0texpecterr0r.elapse-asm'
```

### 使用

在需要进行插桩的方法上加入注解：

```java
@TrackMethod(tag = TAG_LOG)
public void test() {
    try {
        Thread.sleep(1200);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

在 Application 中注册 `MethodObserver`，并实现对应方法：

```java
MethodEventManager.getInstance()
        .registerMethodObserver(TAG_LOG, new MethodObserver() {
            @Override
            public void onMethodEnter(String tag, String methodName) {
                Log.d("MethodEvent", "method "+methodName+" entered at "+System.currentTimeMillis());
            }
            @Override
            public void onMethodExit(String tag, String methodName) {
                Log.d("MethodEvent", "method "+methodName+" exited at "+System.currentTimeMillis());
            }
        });
```

程序运行时，在调用对应方法前后会执行 `MethodObserver` 中的对应方法，从而实现在方法调用前后执行指定逻辑。