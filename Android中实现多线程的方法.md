title: Android中实现多线程的方法
date: 2016-05-24 14:49:41
tags: [Android, 并发]
categories: 杂七杂八

---

在Android中，所有耗时操作都不能放在主线程（即UI线程）中执行，否则会引起ANR异常，进而导致应用程序崩溃。解决办法是使用多线程，将耗时操作放到异步子线程中执行。

Android中实现多线程有两种基本方法：
**1. Thread + Handler**
**2. AsyncTask**

<!--more-->

其中AsyncTask可以视为是对Thread + Handler机制的再次封装实现。**一般来说，数据简单、耗时较短的单个后台异步处理使用AsyncTask，此方法实现代码较为简单，而且更新UI控件更为简单；而执行时间长且逻辑复杂的多后台任务则使用Thread + Handler，此方法与AsyncTask相比能更好的利用系统资源，但是和UI线程的通信会更为复杂。**

Android系统推荐使用AsyncTask来处理简单的异步操作，**通常是在Activity中通过内部类的方式来使用AsyncTask，若不使用内部类，则无法直接更新UI控件，这就失去了使用AsyncTask的意义，不如直接使用Thread + Handler机制来实现多线程。**

在Android中，线程池是有限的，创建过多的异步线程会导致ANR异常，除此之外，使用AsyncTask还需要注意以下[这些问题](http://ruikye.com/2014/08/28/AsyncTask-%E7%9A%84%E4%BD%BF%E7%94%A8/)：

> 1. 由于 AsyncTask 内部的线程池是 static 类型，整个进程共用一个线程池；如果使用不当，会产生阻塞问题，尤其是单任务顺序执行的情况下，一个任务执行时间过长会阻塞其他任务的执行
> 2. static 线程另外一个问题是，如果第一次调用 AsyncTask 在非 UI 线程中，那么以后使用 AsyncTask时，onPosExecute 也会在非 UI 线程中，此时如果执行 UI 操作会 Crash，所以第一次使用 AsyncTask 一定要在 UI 线程中使用，尤其是使用第三方 SDK 时要注意这点
> 3. 通常使用 AsyncTask 是在 Activity 中使用匿名的内部类来使用，内部类的一个问题是会保持外部类的实例，如果 AsyncTask 中的异步任务在 Activity 退出时还没执行完或者阻塞了，那么这个保持的外部的 Activity 实例得不到释放，会引起 OOM 问题，解决办法是：在 AsyncTask 使用弱引用外部实例，或者保证在 Activity 退出时，所有的 AsyncTask 已执行完成或被取消

关于AsyncTask导致的内存泄漏问题，可参考下面这篇文章：

> [Android内存泄漏－AsyncTask的不正确使用](http://huxian99.github.io/2015/09/15/Android%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%EF%BC%8DAsyncTask%E7%9A%84%E4%B8%8D%E6%AD%A3%E7%A1%AE%E4%BD%BF%E7%94%A8/)