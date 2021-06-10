title: 在Android Studio中使用Lambda表达式
weburl: 在Android Studio中使用Lambda表达式
date: 2016-09-26 10:53:34
tags: [Android, Java]
categories: 编程之法

---

Lambda表达式是Java 8中的一个重要新特性，使用Lambda表达式可以简化很多代码，比如常见的为一个按键添加点击事件`Listener`的代码：

```
btn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        // do what you want...
    }
});
```

其中有用的代码很有可能只有一行，其他部分其实都没有实际意义，只是语法需要，创建了一个匿名类而已。但是，如果使用Lambda表达式之后，上述代码就可以简化为：

```
btn.setOnClickListener(v -> {
    // do what you want...
});
```

<!--more-->

如果只有一行代码还可以进一步简化为：

```
btn.setOnClickListener(v -> doSomething());
```

可以看到，使用了Lambda表达式后可以让程序代码更加优雅简洁。关于Java 8中的Lambda表达式详情，可参考以下几篇文章：

> [Java8 lambda表达式10个示例](http://www.importnew.com/16436.html)
> [Java 8新特性：lambda表达式](http://www.liaoxuefeng.com/article/001411306573093ce6ebcdd67624db98acedb2a905c8ea4000)

然而遗憾的是，Android Studio中的Java版本被限制在了Java 1.7以下，与系统中安装的JDK版本无关，也就是就算安装了JDK 1.8也无法使用Java 8的新特性。不过我们依然可以通过一些方法来使用Lambda表达式的，主要方法有两种，下面将分别介绍。

## **RetroLambda** ##
[RetroLambda](https://github.com/orfjackal/retrolambda)是一个Gradle插件，用于实现让低版本的Java（Java 1.7、1.6、1.5等）支持Lambda表达式。做所周知，Android Studio使用的就是Gradle，Gradle Retrolambda插件的官方Github网站及使用说明见[这里](https://github.com/evant/gradle-retrolambda)。在Android Studio中的使用步骤如下：

1.安装[JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。

2.在**项目的**`build.gradle`文件中添加以下代码：
```
buildscript {
  repositories {
     mavenCentral()
  }

  dependencies {
     classpath 'me.tatarka:gradle-retrolambda:3.3.0'
  }
}

// Required because retrolambda is on maven central
repositories {
  mavenCentral()
}
```

3.在**工程模块的**`build.gradle`文件中添加以下代码：
```
apply plugin: 'me.tatarka.retrolambda'

android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

4.在`proguard-rules.pro`文件中添加以下代码：
```
# For retrolambda  
-dontwarn java.lang.invoke.*  
```

5.还可以对Retrolambda进行一些配置，具体见[Github页面的说明](https://github.com/evant/gradle-retrolambda#configuration)

**注意事项：**
retrolambda的版本这里选择了3.3.0，这是目前的最新版本，不过之后肯定会有新版本的，可以根据Github上的说明替换为最新版本。

配置完成后第一次编译时，Android Studio中的Gradle不能配置为Offline模式，否则会提示错误：
```
Gradle sync failed: No cached version of me.tatarka:gradle-retrolambda:3.3.0 available for offline mode.
```

因为上述配置只是指明了需要使用`me.tatarka:gradle-retrolambda:3.3.0`，此时还需要联网下载retrolambda，若配置为离线模式是无法正常下载的。不过只要编译成功一次之后，retrolambda就会被下载到缓存中，此时就可以了改回Offline模式了。

## **Jack** ##
Jack是Java Android Compile Kit的缩写，它是Google为Android推出的一个全新的Java编译工具链。关于Jack的详细信息可参考以下链接：

> [Android 新一代编译 toolchain Jack & Jill 简介](http://taobaofed.org/blog/2016/05/05/new-compiler-for-android/)
> [Compiling with Jack](https://source.android.com/source/jack.html)

使用Jack也是Google官方给出的支持Java 8特性的方法：[Java 8 语言功能](https://developer.android.com/guide/platform/j8-jack.html#supported-features)

要在Android Studio中使用Jack并开启Java 8，只需要在**工程模块的**`build.gradle`文件中添加以下代码：

```
android {
  defaultConfig {
      jackOptions {
          enabled true
      }
  }

  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

使用Jack后，除了支持Lambda表达式外，还可以支持更多的Java 8新特性。不过实际测试下来，使用Jack有两个很大的缺陷：

首先，Jack的编译速度明显要慢很多，而且似乎每次都要从头全部重新编译一次，同样的工程编译所需的时间是原来的5~10倍；

其次，使用Jack后就无法使用[Instant Run](https://developer.android.com/studio/run/index.html#instant-run)功能了，Instant Run是一个很好用的功能，在修改程序后重新推送到设备运行时不需要再次安装apk文件，基本马上就可以看到更改后的变化，这可以大幅提升工作效率。然而目前版本的Jack并没有支持Instant Run。

----------

考虑到Jack这两个缺陷实在会严重降低工作效率，在目前看来使用Jack还不是一个很好的选择。如果要使用Lambda表达式，RetroLambda是一个更好的选择。

