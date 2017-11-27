# AlarmManager学习笔记
## 概述

AlarmManager 提供对系统闹钟服务（或称为定时器服务）的访问接口，使用它既可以指定单次执行的定时任务，也可以指定重复运行的任务。当闹钟指定触发时间到达时，实际上是系统发出为这个闹钟注册的广播，因此我们需要实现一个针对特定闹钟事件的广播接收器(PendingIntent)。

## 注意

从API 19开始，AlarmManager的机制都是非准确传递，操作系统将会转换闹钟，来最小化唤醒和电池使用

## 常用方法

  1. 设置一次性闹钟
  ```java
  void set(int type,long startTime,PendingIntent pi)

  解析：
      type        :   备注1
      startTime   ：  闹钟执行时间
      pi          ：  响应事件
  ```

  2. 周期性执行的定时服务

  ```java
  void setRepeating(int type,long startTime,long intervalTime,PendingIntent pi)

  解析：
      type        :   备注1
      startTime   ：  闹钟执行时间
      intervalTime:   间隔时间（可使用内部定义值 见备注2）
      pi          ：  响应事件
  ```

  3. 周期性执行的定时服务（间隔时间不确定）
  ```java
  void setInexactRepeating(int type,long startTime,long intervalTime,PendingIntent pi)

  解析：
      type        :   备注1
      startTime   ：  闹钟执行时间
      intervalTime:   间隔事件（可使用内部定义值 见备注2）
      pi          ：  响应事件

  说明：
      与第二个方法相似，不过其两个闹钟执行的间隔时间不是固定的而
      已。它相对而言更省电（power-efficient）一些，因为系统可能
      会将几个差不多的闹钟合并为一个来执行，减少设备的唤醒次数.
  ```
  4. 取消定时服务

  ```java
  void cancel(PendingIntent pi)

  说明：
      取消和PendingIntent配置的闹钟服务

  个人观点：
      个人任务取消的应该和PendingIntent 的requestCode 参数有关。
      例如：PendingIntent.getBroadcast(Context context,
      int requestCode, Intent intent, @Flags int flags)
  ```

  ## 备注
    * 备注 1

      *  AlarmManager.ELAPSED_REALTIME在指定的延时过后，发送广播，但不唤醒设备（闹钟在睡眠状态下不可用）。如果在系统休眠时闹钟触发，它将不会被传递，直到下一次设备唤醒。

      * AlarmManager.ELAPSED_REALTIME_WAKEUP
      在指定的延时过后，发送广播，并唤醒设备（即使关机也会执行operation所对应的组件） 。
      延时是要把系统启动的时间SystemClock.elapsedRealtime()算进去的，具体用法看代码。

      * AlarmManager.RTC
      指定当系统调用System.currentTimeMillis()方法返回的值与triggerAtTime相等时启动operation所对应的设备（在指定的时刻，发送广播，但不唤醒设备）。如果在系统休眠时闹钟触发，它将不会被传递，直到下一次设备唤醒（闹钟在睡眠状态下不可用）。

      * AlarmManager.RTC_WAKEUP
      指定当系统调用System.currentTimeMillis()方法返回的值与triggerAtTime相等时启动operation所对应的设备（在指定的时刻，发送广播，并唤醒设备）

    * 备注 2

      * AlarmManager.INTERVAL_FIFTEEN_MINUTES 间隔15分钟
      * AlarmManager.INTERVAL_HALF_HOUR 间隔半个小时
      * AlarmManager.INTERVAL_HOUR 间隔一个小时
      * AlarmManager.INTERVAL_HALF_DAY 间隔半天
      * AlarmManager.INTERVAL_DAY 间隔一天

    ###  FAQ

      * Q1. 设定多个定时闹钟为什么只有最后一个生效

      AlarmManager 根据 PendingIntent requestCode 来判断是否是同一个定时服务，所以当
      requestCode相等的时候只有最后一个生效
