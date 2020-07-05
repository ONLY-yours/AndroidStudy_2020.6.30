# EventBus

### 01前言

​	当我们进行项目开发的时候，往往是需要应用程序的各组件、组件与后台线程间进行通信，比如在子线程中进行请求数据，当数据请求完毕后通过Handler或者是广播通知UI，而两个Fragment之家可以通过Listener进行通信等等。当我们的项目越来越复杂，使用Intent、Handler、Broadcast进行模块间通信、模块与后台线程进行通信时，代码量大，而且高度耦合。现在就让我们来学习一下EventBus 3.0吧。

### 02什么是EventBus

[EventBus Github地址](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgreenrobot%2FEventBus)

​	由greenrobot 组织贡献(该组织还贡献了greenDAO),一个Android事件发布/订阅轻量级框架,

功能:通过解耦发布者和订阅者简化Android事件传递 

​	EventBus可以代替Android传统的Intent,Handler,Broadcast或接口函数,在Fragment,Activity,Service线程之间传递数据，执行方法。

​	特点:代码简洁，是一种发布订阅设计模式(观察者设计模式)。

大概的意思就是：**EventBus能够简化各组件间的通信，让我们的代码书写变得简单，能有效的分离事件发送方和接收方(也就是解耦的意思)，能避免复杂和容易出错的依赖性和生命周期问题。**

### 03 关于EventBus的概述

![image-20200703201219328](C:\Users\I M YOURS\AppData\Roaming\Typora\typora-user-images\image-20200703201219328.png)

#### 三要素

- **Event 事件**。它可以是任意类型。
- **Subscriber 事件订阅者**。在EventBus3.0之前我们必须定义以onEvent开头的那几个方法，分别是onEvent、onEventMainThread、onEventBackgroundThread和onEventAsync，而在3.0之后事件处理的方法名可以随意取，不过需要加上注解@subscribe()，并且指定线程模型，默认是POSTING。
- **Publisher 事件的发布者**。我们可以在任意线程里发布事件，一般情况下，使用EventBus.getDefault()就可以得到一个EventBus对象，然后再调用post(Object)方法即可。

### 四种线程模型

EventBus3.0有四种线程模型，分别是：

- POSTING (默认) 表示事件处理函数的线程跟发布事件的线程在同一个线程。
- MAIN 表示事件处理函数的线程在主线程(UI)线程，因此在这里不能进行耗时操作。
- BACKGROUND 表示事件处理函数的线程在后台线程，因此不能进行UI操作。如果发布事件的线程是主线程(UI线程)，那么事件处理函数将会开启一个后台线程，如果果发布事件的线程是在后台线程，那么事件处理函数就使用该线程。
- ASYNC 表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作。

### 04 EventBus的基本用法

- 添加依赖``implementation 'org.greenrobot:eventbus:3.2.0'``

- 举个例子，我需要在一个Activity里注册EventBus事件，然后定义接收方法，这跟Android里的广播机制很像，你需要首先注册广播，然后需要编写内部类，实现接收广播，然后操作UI。所以，在EventBus中，你同样得这么做。

- 自定义一个事件类（这里有些同学，会有一些疑问，为什么要建立这样一个类，有什么用途。其实这个类就是一个Bean类，里面定义用来传输的数据的类型。）

```java
public class MessageEvent{
    private String message;
    public  MessageEvent(String message){
        this.message=message;
    }
    public String getMessage() {
        return message;
    }
 
    public void setMessage(String message) {
        this.message = message;
    }
}
```

- 注册事件

  ```css
  @Override
  protected void onCreate(Bundle savedInstanceState) {           
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main)；
       EventBus.getDefault().register(this)；
  } 
  ```

  当我们需要在Activity或者Fragment里订阅事件时，我们需要注册EventBus。我们一般选择在Activity的onCreate()方法里去注册EventBus，在onDestory()方法里，去解除注册。

- 解除注册

  ```java
  @Override
  protected void onDestroy() {
      super.onDestroy();
      EventBus.getDefault().unregister(this);
  }
  ```

#### 发送事件

```css
EventBus.getDefault().post(messageEvent);
```

#### 处理事件

```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void XXX(MessageEvent messageEvent) {
    ...
}
```

前面我们说过，处理消息的方法名字可以随便取。但是需要加一个注解@Subscribe，并且要指定线程模型。

### 05粘性事件

- 何为黏性事件呢？简单讲，就是在发送事件之后再订阅该事件也能收到该事件。Android中就有这样的实例，也就是Sticky Broadcast，即粘性广播。正常情况下如果发送者发送了某个广播，而接收者在这个广播发送后才注册自己的Receiver，这时接收者便无法接收到 刚才的广播，为此Android引入了StickyBroadcast，在广播发送结束后会保存刚刚发送的广播（Intent），这样当接收者注册完 Receiver后就可以接收到刚才已经发布的广播。这就使得我们可以预先处理一些事件，让有消费者时再把这些事件投递给消费者.

- EventBus也提供了这样的功能，有所不同是EventBus会存储所有的Sticky事件，如果某个事件在不需要再存储则需要手动进行移除。用户通过Sticky的形式发布事件，而消费者也需要通过Sticky的形式进行注册，当然这种注册除了可以接收 Sticky事件之外和常规的注册功能是一样的，其他类型的事件也会被正常处理。

总结：就是在发送事件的时候，处理的activity或者fragment并没有注册，所以直接发送是收不到的，我们将该事件定义为粘性事件，那么就会放入一个存储的地方，等到需要的地方注册结束后可以第一时间拿到。

**使用:**

1、粘性事件的发布:



```css
EventBus.getDefault().postSticky("RECOGNIZE_SONG");
```

2.粘性事件的接收



```csharp
@Subscribe(threadMode = ThreadMode.MAIN, sticky = true)
    public void receiveSoundRecongnizedmsg(String insType) {
        if ("RECOGNIZE_SONG".equals(insType)) {
            soundRecognizeCtrl();
        }
    }
```

剩下的操作就和普通事件一样注册和反注册即可.

**手动获取和移除粘性事件（Getting and Removing sticky Events manually）**
 正如你之前看到的，最近发布的粘性事件在其新订阅者注册后将会自动传递给新订阅者。但有时可能更方便手动检查粘性事件。有时我们也需要移除粘性事件，以免它在传递下去。

```java
MessageEvent stickyEvent = EventBus.getDefault().getStickyEvent(MessageEvent.class);
if(stickyEvent != null) {
    EventBus.getDefault().removeStickyEvent(stickyEvent);
　　//TODO
}
```

removeStickyEvent 会返回之前持有的粘性事件。于是,

```java
MessageEvent stickyEvent = EventBus.getDefault().removeStickyEvent(MessageEvent.class);
if(stickyEvent != null) {
　　//TODO
}
```

**使用场景**
 我们要把一个Event发送到一个还没有初始化的Activity/Fragment，即尚未订阅事件。那么如果只是简单的post一个事件，那么是无法收到的，这时候，你需要用到粘性事件,它可以帮你解决这类问题.



#### 总结

经过这个简单的例子，我们发现EventBus使用起来是如此的方便，当我们的代码量变得很多的时候，使用EventBus后你的逻辑非常的清晰，并且代码之间高度解耦，在进行组件、页面间通信的时候，EventBus是一个不错的选择。

