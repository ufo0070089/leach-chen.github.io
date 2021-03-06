---
layout: post
title: Android摘记
date:  2018-05-20 16:39:00 +0900  
description: Android摘记
img: post-9.jpg # Add image post (optional)
tags: [Android]
author: # Add name author (optional)
androidoriginal: true
---

## Activity ##

### **1. Activity生命周期** ###

<a href="https://www.cnblogs.com/lwbqqyumidi/p/3769113.html" style="text-decoration: none;" target="\_blank"  title="">onCreate</a>->(onRestart)->onStart->onResume->Activity is run->onPause->onStop->onDestroy

### **2. Fragment生命周期** ###

<a href="https://blog.csdn.net/connor_yang/article/details/75579417" style="text-decoration: none;" target="\_blank"  title="">onAttatch</a>->onCreate->onCreateView->onActivityCreated->onStart->onResume->Fragment is active->onPause->onStop->onDestroyView->onDestroy->onDetach

这张图是Fragment生命周期和Activity生命周期对比图，可以看到两者还是有很多相似的地方，比如都有onCreate()，onStart()，onPause()，onDestroy()等等，因为Fragment是被托管到Activity中的，所以多了两个onAttach()和onDetach()。这里讲讲与Activity生命周期不一样的方法。

onAttach()，Fragment和Activity建立关联的时候调用，被附加到Activity中去。

onCreate()，系统会在创建Fragment时调用此方法。可以初始化一段资源文件等等。

onCreateView()，系统会在Fragment首次绘制其用户界面时调用此方法。 要想为Fragment绘制 UI，从该方法中返回的 View 必须是Fragment布局的根视图。如果Fragment未提供 UI，您可以返回 null。

onViewCreated()，在Fragment被绘制后，调用此方法，可以初始化控件资源。

onActivityCreated()，当onCreate()，onCreateView()，onViewCreated()方法执行完后调用，也就是Activity被渲染绘制出来后。

onPause()，系统将此方法作为用户离开Fragment的第一个信号（但并不总是意味着此Fragment会被销毁）进行调用。 通常可以在此方法内确认在当前用户会话结束后仍然有效的任何更改（因为用户可能不会返回）。

onDestroyView()，Fragment中的布局被移除时调用。

onDetach()，Fragment和Activity解除关联的时候调用。

但需要注一点是：除了onCreateView，其他的所有方法如果你重写了，必须调用父类对于该方法的实现。

### **3. Activity的几种LaunchMode及使用场景** ###

**a.standard：** 标准模式，系统默认模式。每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。在这个模式下，谁启动了Activity，那么这个Activity就运行在启动它的那个Activity所在栈中。

**b.singleTop：** 栈顶复用模式。在这种模式下，如果新的Activity已经位于任务栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被回调，通过此方法的参数我们可以取出当前的请求信息

**c.singleTask：** 栈内复用模式。这是一种单例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会创建实例，和singleTop是一样，系统也会调用onNewIntent。还有一点，就是singleTask有clearTop的效果，会导致栈内已有的Activity全部出栈。

**d.singleInstance：** 单一实例模式。这是一种加强的singleTask模式，它除了具有singleTask的所有特性以外，还加强了一点，那就是具有此模式的Activity只能单独位于一个任务栈中，比如Activity A是singleInstance模式，当A启动后，系统会为它创建一个新的任务栈，然后A独自在这个新的任务栈中，由于栈内复用的特性，后续均不会创建新的Activity，除非这个独特的任务栈被系统销毁。整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity 共享公用的同一个activity。他会运行在自己单独，独立的任务栈里面，并且任务栈里面只有他一个实例存在。

使用场景：

**a.standard使用场景（普通）：** 邮件客户端，在新建一个邮件的时候，适合新建一个新的实例

**b.singleTop使用场景（通知）：** 消息推送，通知栏弹出Notification，点击Notification跳转到指定Activity，使用singleTop避免生成重复的页面。登录的时候，登录成功跳转到主页，按下两次登录按钮，使用singleTask避免生成两个主页。从activity A启动了个service进行耗时操作，或者某种监听，这个时候你home键了，service收集到信息，要返回activityA。

**c.singleTask使用场景（启动主界面）：** 提供给第三方应用调用的页面，做浏览器、微博之类的应用，浏览器的主界面等等。程序的主界面，进入多层嵌套之后，一键退回，之前打开的Activity全部出栈。

**d.singleInstance使用场景（被多个应用公共调用的如别人可以调用你的应用进行分享）：** 呼叫来电界面，打电话、发短信功能。闹铃提醒，将闹铃提醒与闹铃设置分离。


## View ##

### **1. View的绘制流程** ###

**a. measure过程**

<a href="https://blog.csdn.net/yanbober/article/details/46128379/" style="text-decoration: none;" target="\_blank"  title="">通过</a>上面分析可以看出measure过程主要就是从顶层父View向子View递归调用view.measure方法（measure中又回调onMeasure方法）的过程。具体measure核心主要有如下几点：

MeasureSpec（View的内部类）测量规格为int型，值由高2位规格模式specMode和低30位具体尺寸specSize组成。其中specMode只有三种值：

1.MeasureSpec.EXACTLY//确定模式，父View希望子View的大小是确定的，由specSize决定；2.MeasureSpec.AT_MOST//最多模式，父View希望子View的大小最多是specSize指定的值；3.MeasureSpec.UNSPECIFIED//未指定模式，父View完全依据子View的设计值来决定；

View的measure方法是final的，不允许重载，View子类只能重载onMeasure来完成自己的测量逻辑。

最顶层DecorView测量时的MeasureSpec是由ViewRootImpl中getRootMeasureSpec方法确定的（LayoutParams宽高参数均为MATCH_PARENT，specMode是EXACTLY，specSize为物理屏幕大小）。

ViewGroup类提供了measureChild，measureChild和measureChildWithMargins方法，简化了父子View的尺寸计算。

只要是ViewGroup的子类就必须要求LayoutParams继承子MarginLayoutParams，否则无法使用layout_margin参数。

View的布局大小由父View和子View共同决定。

使用View的getMeasuredWidth()和getMeasuredHeight()方法来获取View测量的宽高，必须保证这两个方法在onMeasure流程之后被调用才能返回有效值。

**b. layout过程**

整个layout过程比较容易理解，从上面分析可以看出layout也是从顶层父View向子View的递归调用view.layout方法的过程，即父View根据上一步measure子View所得到的布局大小和布局参数，将子View放在合适的位置上。具体layout核心主要有以下几点：

View.layout方法可被重载，ViewGroup.layout为final的不可重载，ViewGroup.onLayout为abstract的，子类必须重载实现自己的位置逻辑。

measure操作完成后得到的是对每个View经测量过的measuredWidth和measuredHeight，layout操作完成之后得到的是对每个View进行位置分配后的mLeft、mTop、mRight、mBottom，这些值都是相对于父View来说的。

凡是layout_XXX的布局属性基本都针对的是包含子View的ViewGroup的，当对一个没有父容器的View设置相关layout_XXX属性是没有任何意义的（前面《Android应用setContentView与LayoutInflater加载解析机制源码分析》也有提到过）。

使用View的getWidth()和getHeight()方法来获取View测量的宽高，必须保证这两个方法在onLayout流程之后被调用才能返回有效值。

**c. draw过程**

可以看见，绘制过程就是把View对象绘制到屏幕上，整个draw过程需要注意如下细节：

如果该View是一个ViewGroup，则需要递归绘制其所包含的所有子View。

View默认不会绘制任何内容，真正的绘制都需要自己在子类中实现。

View的绘制是借助onDraw方法传入的Canvas类来进行的。

区分View动画和ViewGroup布局动画，前者指的是View自身的动画，可以通过setAnimation添加，后者是专门针对ViewGroup显示内部子视图时设置的动画，可以在xml布局文件中对ViewGroup设置layoutAnimation属性（譬如对LinearLayout设置子View在显示时出现逐行、随机、下等显示等不同动画效果）。

在获取画布剪切区（每个View的draw中传入的Canvas）时会自动处理掉padding，子View获取Canvas不用关注这些逻辑，只用关心如何绘制即可。

默认情况下子View的ViewGroup.drawChild绘制顺序和子View被添加的顺序一致，但是你也可以重载ViewGroup.getChildDrawingOrder()方法提供不同顺序。

### **2. 事件传递机制** ###

事件首先到父view的distachTouchEvent，返回true事件消费掉了，只到该view的onTouchEvent为止，为false，若父view为viewgroup，则有onInterceptTouchEvent，返回false则事件继续向下传递，若返回true，则拦截事件，最终都是通过通过view或者viewgroup的onTouchEvent往上传，若返回true则该view消费，否则直接到Activity的onTouchEvent事件

如果 View 没有对 ACTION_DOWN 进行消费，之后的其他事件不会传递过来。

OnTouchListener 优先于 onTouchEvent()对事件进行消费。

distachTouchEvent-》onInterceptTouchEvent-》

onTouchEvent《-onTouchEvent《-

### **3. 滑动冲突的处理** ###

1：外部拦截，父容器进行拦截事件处理，如listview里面嵌套viewpager，在viewpager上触摸滑动时，需要在外层listview onInterceptTouchEvent里面判断是水平还是垂直方向滑动，若是垂直，则拦截事件，否则交给viewpager进行滑动

2：内部拦截，父容器不做事件拦截处理，在子控件的dispatchTouchEvent方法里面，判断父控件是否要处理事件，不需要则调用 getParent().requestDisallowInterceptTouchEvent(true)，让父控件跳过事件拦截函数，处理完成后再调用getParent().requestDisallowInterceptTouchEvent(false);如viewpager里面有viewpager，值viewpager滑动完成后就把事件交给父viewpager，父viewpager继续滑动


## 机制原理 ##

### **1. 安卓中消息循环机制如何，Handler通信机制？** ###

主线程创建后会调用Loop.prepareMainLooper(),最终调用Loop.prepare()创建Loop对象，Loop对象创建后还会创建一个MessageQueue，然后会调用Loop.loop()进入一个无限的消息循环，此时若要读或者取消息队列中的消息时，我们可以创建个Handler来处理，一个Handler对应一个Message，当消息循环到时，会通过Message里面保存的Handler对象，回调handleMessage(),handler也可以往消息队列中添加消息，当消息队列中的消息为空时线程就会挂起，一个线程只能调用一次Loop.prepare(),因为内部有判断，当Loop对象存在时，再调用会抛出异常。

### **2. Binder通信机制** ###

Binder是用来解决进程间通信的，它由服务端，驱动层（基于一段内存），客户端组成。服务端需要继承Binder重写onTransact函数，服务端创建的时候需要创建一个服务端的Binder对象，该对象创建的时候，会启动一个隐藏线程，用来接收驱动层发来的消息。在服务端创建的时候，Binder驱动层也会创建一个Binder引用mRemote，如果客户端和服务端在一个进程，则客户端获取到的Binder是服务端的这个Binder，若在不同进程，则获取到的是Binder驱动层的这个Binder mRemote，客户端只有通过bindService方式才能获取到Binder引用，startService就不行了，这样下来就可以进行跨进程通信了。


### **3. APP Launch启动过程** ###

APP是由Launch启动，Launch其实也是个Activity，它随系统启动，会加载所有APP信息，信息主要来至清单文件，并根据清单文件信息创建图标并设置点击事件。若APP是第一次启动，则AMS会启动新的进程，并在新进程中创建ActivityThread对象，执行main方法，APP主线程启动后将applicationThread传给AMS,AMS通知绑定Application，并启动默认要启动的Activity

### **4. Activity启动流程** ###

在新进程中启动activity总体可分为5个阶段:

a.启动进程调用startActivity()

b.AMS做一系列的准备工作:

   1) 解析intent并保存在activityInfo中

   2) 如果有多个activity满足要求,则弹出对话框让用户选择

   3) 根据启动模式进行任务栈的操作,找出或创建activity应该属于的任务栈,并将activity放到栈顶

   4) 暂停当前显示的activity

   5) 创建新进程

c.执行新建进程的ActivityThread.main()方法, 然后attach到AMS,并将新进程的ApplicationThread对象传给AMS.该对象是个binder代理对象.

d.AMS再做几件事:

   1) 给目标进程的ApplicationThread对象注册binder死亡通知

   2) 创建目标进程的ProcessRecord.

   3) 初始化目标进程的运行环境

e.进入目标进程创建并显示activity

   1) 加载activity实例,并创建application,contextImpl和config等对象.

   2) 创建与activity关联的PhoneWindow和WindowManagerImpl对象

   3) 回调activity的onCreate()和onStart()方法

   4) 在onCreate()方法中调用setContentView()方法创建activity的根视图DecorView并加载layout文件到DecorView.

   5) 回调activity的onResume()方法,然后创建ViewRootImpl并调用setView方法进行界面的绘制

   6) 创建与InputManager的连接以便接收键盘和触摸屏事件.


## 性能优化 ##

### **1. 内存泄露出现情况，如何排查优化** ###

出现情况（异步任务还没结束，界面就退出了）：

1：Handler延迟执行，退出的时候没有remove掉Runnable

2：非静态内部类里面有线程，里面有sleep

3：viewpager 提前把所有view创建好

4：不用的资源没有及时关闭或释放，如Bitmap

优化（界面退出的时候要及时停止掉异步任务）：

1：使用静态内部类或者对象使用弱引用

2：viewpager 在instantiateItem里面创建view，不要提前把所有view创建好

3：不用的资源要及时关闭，如打开文件，数据库，图片对象置空等

4：Android Studio里面的内存、CPU工具排查，抓取到可能存在问题的代码段，然后分析该处逻辑是否有缺陷

5：进行代码审查


## 动画 ##

### **1. 动画类型** ###

有补间动画（平移、缩放、渐变、旋转）、属性动画（插值器：决定变化的规律，如匀速加速，估值器；计算具体的值，如0-1变化，估值器计算值时依赖插值器返回的值来从0变化到1），material design视觉差，第三方库，自定义view，转场动画（设置共享元素）


## Service ##

### **1. Service生命周期** ###

这里要注意service有两种启动方式，startService()和bindService()

onCreate() ——> onStartCommand() ——> Service running  ——>onDestroy()  ——> Service stop

context.bindService()  ——> onCreate()  ——> onBind()(只一次，不可多次绑定)  ——> Service running  ——> onUnbind()  ——> onDestroy()  ——> Service stop

多次调用startService，该Service只能被创建一次，即该Service的onCreate方法只会被调用一次。但是每次调用startService，onStartCommand方法都会被调用。Service的onStart方法在API 5时被废弃，替代它的是onStartCommand方法。

第一次执行bindService时，onCreate和onBind方法会被调用，但是多次执行bindService时，onCreate和onBind方法并不会被多次调用，即并不会多次创建服务和绑定服务。

### **2. IntentService** ###

IntentService是继承并处理异步请求的一个类，在IntentService内有一个工作线程来处理耗时操作，启动IntentService的方式和启动传统的Service一样，同时，当任务执行完后，IntentService会自动停止，而不需要我们手动去控制或stopSelf()。另外，可以启动IntentService多次，而每一个耗时操作会以工作队列的方式在IntentService的onHandleIntent回调方法中执行，并且，每次只会执行一个工作线程，执行完第一个再执行第二个，以此类推。




## 其它 ##


RxJava原理，Retrofit原理，GreenDao原理，OkHttp原理

数据结构

算法
