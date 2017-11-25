# JobScheduler学习笔记

先上代码如下：
```java
/**
 * Created by Li on 2017/11/2.
 */
@TargetApi(21)
public class JobSchedule extends JobService{
    @Override
    public boolean onStartJob(JobParameters params) {
        //遇到一个问题：当我使用okhttp异步请求数据时，doInBackground函数返回不了数据,改为同步时可以
        final AsyncTask<JobParameters,Void,String> task= new AsyncTask<JobParameters,Void,String>(){
            private JobParameters parameters;
            private OkHttpClient okHttpClient;
            String conten;
            @Override
            protected String doInBackground(JobParameters... params) {
                parameters=params[0];
                okHttpClient=new OkHttpClient();
                Request request=new Request.Builder().url("http://www.baidu.com").build();
                Call call=okHttpClient.newCall(request);
                try {
                    Response execute= call.execute();
                    conten=  execute.body().string();
                    Log.i("JobSchedule"," conten "+conten);
                } catch (IOException e) {
                    e.printStackTrace();
                }

                return conten;

            }

            @Override
            protected void onPostExecute(String s) {


                Log.i("JobSchedule","ID  "+s);
                super.onPostExecute(s);
            }
        };



        task.execute(params);
        jobFinished(params,true);
        return true;
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        return false;
    }
}
```
## 什么是JobSchedule（必要性）

在日常开发中，有时候我们想要让任务在稍后的某个时间点或者当达到一定的条件时开始执行，当预设的条件都满足时，就执行我们的程序，这里注意一点，我们的执行时间是不确定的，也就是说，我们不知道什么时候开始执行程序。

### （1）创建JobService

新建一个Java类，继承JobService，这个类必须要实现两个方法，如下：
```java

public boolean onStartJob(JobParameters params);

public boolean onStopJob(JobParameters params);
```
当任务开始时会执行onStartJob(JobParameters params)方法,因为这是系统用来触发已经被执行的任务。正如你所看到的，这个方法返回一个boolean值。如果返回值是false,系统假设这个方法返回时任务已经执行完毕。如果返回值是true,那么系统假定这个任务正要被执行，执行任务的重担就落在了你的肩上。当任务执行完毕时你需要用jobFinished(JobParameters params, boolean needsRescheduled)来通知系统。

当系统接收到一个取消请求时，系统会调用`onStopJob(JobParameters params)`方法取消正在等待执行的任务。很重要的一点是如果`onStartJob(JobParameters params)`返回false,那么系统假定在接收到一个取消请求时已经没有正在运行的任务。换句话说，`onStopJob(JobParameters params)`在这种情况下不会被调用。

需要注意的是这个job service运行在你的主线程，这意味着你需要使用子线程，handler, 或者一个异步任务来运行耗时的操作以防止阻塞主线程。

当任务执行完毕之后，你需要调用jobFinished(JobParameters params, boolean needsRescheduled)来让系统知道这个任务已经结束，系统可以将下一个任务添加到队列中。如果你没有调用jobFinished(JobParameters params, boolean needsRescheduled)，你的任务只会执行一次，而应用中的其他任务就不会被执行。

jobFinished(JobParameters params, boolean needsRescheduled)的两个参数中的params参数是从JobService的onStartJob(JobParameters params)的params传递过来的，needsRescheduled参数是让系统知道这个任务是否应该在最处的条件下被重复执行。这个boolean值很有用，因为它指明了你如何处理由于其他原因导致任务执行失败的情况，例如一个失败的网络请求调用。

一旦你在Java部分做了上述工作之后，你需要到AndroidManifest.xml中添加一个service节点让你的应用拥有绑定和使用这个JobService的权限。
```java
<service
    android:name=".com.lans.lwk.appclass.JobSchedule"
    android:permission="android.permission.BIND_JOB_SERVICE" />
```

### (2)创建一个JobScheduler对象
第一件要做的事就是你需要创建一个JobScheduler对象，在实例代码的MainActivity中我们通过getSystemService(Context.JOB_SCHEDULER_SERVICE )初始化了一个叫做jobScheduler的JobScheduler对象。

```java
 JobScheduler jobScheduler=(JobScheduler)getSystemService(Context.JOB_SCHEDULER_SERVICE);
```

当你想创建定时任务时，你可以使用JobInfo.Builder来构建一个JobInfo对象，然后传递给你的Service。JobInfo.Builder接收两个参数，第一个参数是你要运行的任务的标识符，第二个是这个Service组件的类名。

```java
  componentName=new ComponentName(this,JobSchedule.class);
  jobInfo=new JobInfo.Builder(i,componentName).setPeriodic(3000).build();
```

这个builder允许你设置很多不同的选项来控制任务的执行。下面的代码片段就是展示了如何设置以使得你的任务可以每隔三秒运行一次。

**其他设置方法 :**

  * setMinimumLatency(long minLatencyMillis): 这个函数能让你设置任务的延迟执行时间(单位是毫秒),这个函数与setPeriodic(long time)方法不兼容，如果这两个方法同时调用了就会引起异常；

  * setOverrideDeadline(long maxExecutionDelayMillis):
这个方法让你可以设置任务最晚的延迟时间。如果到了规定的时间时其他条件还未满足，你的任务也会被启动。与setMinimumLatency(long time)一样，这个方法也会与setPeriodic(long time)，同时调用这两个方法会引发异常。
  * setPersisted(boolean isPersisted):
这个方法告诉系统当你的设备重启之后你的任务是否还要继续执行。

  * setRequiredNetworkType(int networkType):
这个方法让你这个任务只有在满足指定的网络条件时才会被执行。默认条件是JobInfo.NETWORK_TYPE_NONE，这意味着不管是否有网络这个任务都会被执行。另外两个可选类型，一种是JobInfo.NETWORK_TYPE_ANY，它表明需要任意一种网络才使得任务可以执行。另一种是JobInfo.NETWORK_TYPE_UNMETERED，它表示设备不是蜂窝网络( 比如在WIFI连接时 )时任务才会被执行。

  * setRequiresCharging(boolean requiresCharging):
这个方法告诉你的应用，只有当设备在充电时这个任务才会被执行。

  * setRequiresDeviceIdle(boolean requiresDeviceIdle):
这个方法告诉你的任务只有当用户没有在使用该设备且有一段时间没有使用时才会启动该任务。

__需要注意的是__setRequiredNetworkType(int networkType), setRequiresCharging(boolean requireCharging) and setRequiresDeviceIdle(boolean requireIdle)者几个方法可能会使得你的任务无法执行，除非调用setOverrideDeadline(long time)设置了最大延迟时间，使得你的任务在为满足条件的情况下也会被执行。一旦你预置的条件被设置，你就可以构建一个JobInfo对象，然后通过如下所示的代码将它发送到你的JobScheduler中。

```java
int res=jobScheduler.schedule(jobInfo);
```
你可能注意到了，这个schedule方法会返回一个整型。如果schedule方法失败了，它会返回一个小于0的错误码。否则它会我们在JobInfo.Builder中定义的标识id。

如果你的应用想停止某个任务，你可以调用JobScheduler对象的cancel(int jobId)来实现；如果你想取消所有的任务，你可以调用JobScheduler对象的cancelAll()来实现。
