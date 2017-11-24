# EventBus学习笔记

## 概述

EventBus是针一款对Android的发布/订阅事件总线。它可以让我们很轻松的
实现在Android各个组件之间传递消息，并且代码的可读性更好，耦合度更低。

## 使用

(1) 首先需要定义一个消息类，该类可以不继承任何基类也不需要实现任何接口。
如：
```java
public class EventBusTest {
private String msg;
public EventBusTest(String msg) {
    this.msg=msg;
}

public String getMsg(){
    return  msg;
}
}
```

(2) 在需要订阅事件的地方注册事件
```java
EventBus.getDefault().register(this);
```
(3) 产生事件，即发送消息
```java
EventBus.getDefault().post(new EventBusTest("HELLO from Match"));
```

(4) 处理消息
```java
@Subscribe(threadMode = ThreadMode.MAIN,sticky = true,priority = 1)
 public void OnShowMessage(EventBusTest test){
     setTitle(test.getMsg());
     Toast.makeText(this,test.getMsg(),Toast.LENGTH_LONG).show();

 }
```

(5) 取消消息订阅

```java
EventBus.getDefault().unregister(this);
```

## 优点

解耦，使耦合度降低，并且使代码逻辑清晰

在EventBus的事件处理函数中需要指定线程模型，即指定事件处理函数运行所在的想线程。在上面我们已经接触到了EventBus的四种线程模型。
在EventBus中的观察者通常有四种线程模型，分别是PostIng（默认）、Main、Background与Async。

  * PostING：如果使用事件处理函数指定了线程模型为PostING，那么该事件在
  哪个线程发布出来的，事件处理函数就会在这个线程中运行，也就是说发
  布事件和接收事件在同一个线程。在线程模型为PostING的事件处理函数中
  尽量避免执行耗时操作，因为它会阻塞事件的传递，甚至有可能会引起ANR。

  * Main：如果使用事件处理函数指定了线程模型为Main，那么不论事件
  是在哪个线程中发布出来的，该事件处理函数都会在UI线程中执行。该
  方法可以用来更新UI，但是不能处理耗时操作。

  * Background：如果使用事件处理函数指定了线程模型为Background
，那么如果事件是在UI线程中发布出来的，那么该事件处理函数就会在新
的线程中运行，如果事件本来就是子线程中发布出来的，那么该事件
处理函数直接在发布事件的线程中执行。在此事件处理函数中禁止进
行UI更新操作。

* Async：如果使用事件处理函数指定了线程模型为Async，那么无
论事件在哪个线程发布，该事件处理函数都会在新建的子线程中执行。
同样，此事
件处理函数中禁止进行UI更新操作。

## 黏性事件

订阅黏性事件：
```java
EventBus.getDefault().register(StickyModeActivity.this);
```
黏性事件处理函数：
```java
@Subscribe(sticky = true)
public void XXX(MessageEvent messageEvent) {
    ......
}
```
发送黏性事件：
```java
EventBus.getDefault().postSticky(new MessageEvent("test"));
```
`简单来说粘性事件会在订阅后收到之前发送的消息，但是只会收到最近的一条消息`
