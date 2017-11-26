# WakeLock学习笔记

# 必要性
android系统在手机屏幕锁定之后一般会让手机休眠，以提高电池的使用时间。但是休眠意味着CPU频率降低，有时候可能需要做一些需要大量运算的任务，所以需要唤醒CPU。WakeLock可以做到这一点。

```java
PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);

Wakelock wakeLock=powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,

"MyWakelockTag");
```

另外如果要使用WakeLock需要在Manifest中添加如下权限
```java
<uses-permission android:name="android.permission.WAKE_LOCK" />
```
 * **WakeLock的等级**

 上面的第一个参数是WakeLock levelAndFlag，分别代表了一种WakeLock等级，并且可以通过「或」位操作组合使用。他们是：
  * PARTIAL_WAKE_LOCK：保证CPU保持高性能运行，而屏幕和键盘背光（也可能是触摸按键的背光）关闭。一般情况下都会使用这个WakeLock。
  * ACQUIRE_CAUSES_WAKEUP：这个WakeLock除了会使CPU高性能运行外还会导致屏幕亮起，即使屏幕原先处于关闭的状态下。
  * ON_AFTER_RELEASE：如果释放WakeLock的时候屏幕处于亮着的状态，则在释放WakeLock之后让屏幕再保持亮一小会。如果释放WakeLock的时候屏幕本身就没亮，则不会有动作。

  **被弃用的WakeLock：**

  * SCREEN_DIM_WAKE_LOCK：保证屏幕亮起，但是亮度可能比较低。同时键盘背光也可以不亮。
  * SCREEN_BRIGHT_WAKE_LOCK ：保证屏幕全亮，同时键盘背光也亮。
  * FULL_WAKE_LOCK：表现和SCREEN_BRIGHT_WAKE_LOCK 类似，但是区别在于这个等级的WakeLock使用的是最高亮度！


  上面三个类型的作用无非就是保持屏幕长亮。所以推荐是用WindowFlag` WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON`。使用方法是：

  1. Activity中：

  ```java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
```  
  2. 或者在布局中

  ```java
android:keepScreenOn="true"
  ```

  如果要清除掉这个flag请使用如下
  ```java
getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
  ```

  ## WakeLock的使用
   ### 获取WakeLock：
   ```java
   void acquire()://获得WakeLock
   void acquire(long timeOut)://获得WakeLock timeOut时长，当超过timeOut之后系统自动释放WakeLock。
   ```

   ### 释放WakeLock
   ```java
      release();
   ```
   ### 判断是否已经获取WakeLock
   ```java
boolean isHeld()
   ```
   ### 是否开启引用计数法
   void setReferenceCounted(boolean value)：是否使用引用计数。类似于垃圾回收策略，只是把垃圾回收改成了WakeLock回收。如果value是true的话将启用该特性，如果一个WakeLock acquire了多次也必须release多次才能释放掉。但是如果释放次数比acquire多则会抛出RuntimeException("WakeLock under-locked " + mTag)异常。默认是开启了引用计数的！

  ##  PowerManager的几个实用方法

   * `boolean PowerManager::isScreenOn ()`判断屏幕是否亮着（不管是暗的dimed还是正常亮度），在API20被弃用，推荐`boolean PowerManager::isInteractive ()`

   * `void PowerManager::goToSleep(long time)`time是时间戳，一般是System.currentTimeMillis()+timeDelay。强制系统立刻休眠，需要Manifest中添加权限`"android.permission.DEVICE_POWER"`。按下电源键锁屏时调用的就是这个方法。

   * `void PowerManager::wakeUp(long time)`与上面对应。参数含义，所需权限与上同。按下电源键解锁屏幕时调用的就是这个方法。

   * `void PowerManager::reboot(String reason)`重启手机，reason是要传给linux内核的参数，比如“recovery”重启进recovery模式，“fastboot”重启进fastboot模式。需要权限`"android.permission.REBOOT"。`

   ## 方法二: WakefulBroadcastReceiver

   推荐的方式是使用WakefulBroadcastReceiver：使用广播和Service（典型的IntentService）结合的方式可以让你很好地管理后台服务的生命周期。

   WakefulBroadcastReceiver是BroadcastReceiver的一种特例。它会为你的APP创建和管理一个PARTIAL_WAKE_LOCK 类型的WakeLock。一个WakeBroadcastReceiver接收到广播后将工作传递给Service（一个典型的IntentService），直到确保设备没有休眠。如果你在交接工作给服务的时候没有保持唤醒锁，在工作还没完成之前就允许设备休眠的话，将会出现一些你不愿意看到的情况

   使用WakefulBroadcastReceiver第一步就是在Manifest中注册：
   ```java
<receiver android:name=".MyWakefulReceiver"></receiver>
   ```
   使用`startWakefulService()`方法来启动服务，与`startService()`相比，在启动服务的同时，并启用了唤醒锁。

   ```java
    public class MyWakefulReceiver extends WakefulBroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {        

         Intent service = new Intent(context, MyIntentService.class);        
         startWakefulService(context, service);
    }
    }
      ```

      当后台服务的任务完成，要调用`MyWakefulReceiver.completeWakefulIntent()`来释放唤醒锁。如下:
      ```java
      public class WakeLockService extends IntentService {

          public WakeLockService(String name) {
              super(name);
          }

          @Override
          protected void onHandleIntent(Intent intent) {

              WakefullReceiver.completeWakefulIntent(intent);
          }

      }
      ```
      ## 其他的实现方法

      采用定时重复的Service开启：

　　1、利用Android自带的定时器AlarmManager实现

  ```java
Intent intent = new Intent(mContext, ServiceTest.class);
PendingIntent pi = PendingIntent.getService(mContext, 1, intent, 0);
AlarmManager alarm = (AlarmManager) getSystemService(Service.ALARM_SERVICE);
if(alarm != null)
{
    alarm.cancel(pi);
    // 闹钟在系统睡眠状态下会唤醒系统并执行提示功能
    alarm.setRepeating(AlarmManager.RTC_WAKEUP, System.currentTimeMillis() + 1000, 2000, pi);
    // 确切的时间闹钟//alarm.setExact(…);
    //alarm.set(AlarmManager.RTC_WAKEUP, System.currentTimeMillis(), pi);
}
```
　2、该定时器可以启动Service服务、发送广播、跳转Activity，并且会在系统睡眠状态下唤醒系统。所以该方法不用获取电源锁和释放电源锁。

`注意：在19以上版本，setRepeating中设置的频繁只是建议值(6.0 的源码中最小值是60s)，如果要精确一些的用setWindow或者setExact。`

总结：

1. 关键逻辑的执行过程，就需要Wake Lock来保护。如断线重连重新登陆
2. 休眠的情况下如何唤醒来执行任务？用AlarmManager。如推送消息的获取

### 批量任务优化JobService

```java
import android.app.job.JobInfo;
import android.app.job.JobScheduler;
import android.support.v4.content.WakefulBroadcastReceiver;

public class MainActivity extends AppCompatActivity {
    TextView wakelock_text;
    PowerManager pw;
    PowerManager.WakeLock mWakelock;
    private ComponentName serviceComponent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        wakelock_text = (TextView) findViewById(R.id.wakelock_text);
        pw = (PowerManager) getSystemService(POWER_SERVICE);
        mWakelock = pw.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, "mywakelock");
        serviceComponent = new ComponentName(this,MyJobService.class);
    }

    public void execut(View view) {
        wakelock_text.setText("正在下载....");
//        for (int i = 0; i < 500; i++) {
//            mWakelock.acquire();//唤醒CPU
//            wakelock_text.append(i+"连接中……");
////            wakelock_text.append("");
//            //下载
//            if (isNetWorkConnected()) {
//                new SimpleDownloadTask().execute();
//            } else {
//                wakelock_text.append("没有网络连接。");
//            }
//        }

        //优化
        JobScheduler jobScheduler = (JobScheduler) getSystemService(Context.JOB_SCHEDULER_SERVICE);
        for (int i = 0; i < 500; i++) {
            JobInfo jobInfo = new JobInfo.Builder(i,serviceComponent)
                    .setMinimumLatency(5000)//5秒 最小延时、
                    .setOverrideDeadline(60000)//maximum最多执行时间
//                    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)//免费的网络---wifi 蓝牙 USB
                    .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)//任意网络---wifi
                    .build();
            jobScheduler.schedule(jobInfo);
        }

    }

    private boolean isNetWorkConnected() {
        ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
        return (activeNetworkInfo != null && activeNetworkInfo.isConnected());
    }

    /**
     * Uses AsyncTask to create a task away from the main UI thread. This task creates a
     * HTTPUrlConnection, and then downloads the contents of the webpage as an InputStream.
     * The InputStream is then converted to a String, which is displayed in the UI by the
     * onPostExecute() method.
     */
    private static final String LOG_TAG = "ricky";
    private int index=0;

    private class SimpleDownloadTask extends AsyncTask<Void, Void, String> {

        @Override
        protected String doInBackground(Void... params) {
            try {
                // Only display the first 50 characters of the retrieved web page content.
                int len = 50;
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
//                URL url = new URL("https://www.google.com");
                URL url = new URL("https://www.baidu.com");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setReadTimeout(10000); // 10 seconds
                conn.setConnectTimeout(15000); // 15 seconds
                conn.setRequestMethod("GET");
                //Starts the query
                conn.connect();
                int response = conn.getResponseCode();
                index++;
                Log.d(LOG_TAG,  index+"The response is: " + response);
                InputStream is = conn.getInputStream();

                // Convert the input stream to a string
                Reader reader = new InputStreamReader(is, "UTF-8");
                char[] buffer = new char[len];
                reader.read(buffer);
                return new String(buffer);

            } catch (IOException e) {
                return "Unable to retrieve web page.";
            }
        }

        @Override
        protected void onPostExecute(String result) {
            wakelock_text.append("\n" + result + "\n");
            releaseWakeLock();
        }
    }

    private void releaseWakeLock() {
        if (mWakelock.isHeld()) {
            mWakelock.release();//记得释放CPU锁
            wakelock_text.append("释放锁！");
        }
    }

}
```

```java
public class MyJobService extends JobService {
    private static final String LOG_TAG = "MyJobService";

    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(LOG_TAG, "MyJobService created");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i(LOG_TAG, "MyJobService destroyed");
    }

    @Override
    public boolean onStartJob(JobParameters params) {
        // This is where you would implement all of the logic for your job. Note that this runs
        // on the main thread, so you will want to use a separate thread for asynchronous work
        // (as we demonstrate below to establish a network connection).
        // If you use a separate thread, return true to indicate that you need a "reschedule" to
        // return to the job at some point in the future to finish processing the work. Otherwise,
        // return false when finished.
        Log.i(LOG_TAG, "Totally and completely working on job " + params.getJobId());
        // First, check the network, and then attempt to connect.
        if (isNetworkConnected()) {
            new SimpleDownloadTask() .execute(params);
            return true;
        } else {
            Log.i(LOG_TAG, "No connection on job " + params.getJobId() + "; sad face");
        }
        return false;
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        // Called if the job must be stopped before jobFinished() has been called. This may
        // happen if the requirements are no longer being met, such as the user no longer
        // connecting to WiFi, or the device no longer being idle. Use this callback to resolve
        // anything that may cause your application to misbehave from the job being halted.
        // Return true if the job should be rescheduled based on the retry criteria specified
        // when the job was created or return false to drop the job. Regardless of the value
        // returned, your job must stop executing.
        Log.i(LOG_TAG, "Whelp, something changed, so I'm calling it on job " + params.getJobId());
        return false;
    }

    /**
     * Determines if the device is currently online.
     */
    private boolean isNetworkConnected() {
        ConnectivityManager connectivityManager =
                (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
        return (networkInfo != null && networkInfo.isConnected());
    }

    /**
     *  Uses AsyncTask to create a task away from the main UI thread. This task creates a
     *  HTTPUrlConnection, and then downloads the contents of the webpage as an InputStream.
     *  The InputStream is then converted to a String, which is logged by the
     *  onPostExecute() method.
     */
    private class SimpleDownloadTask extends AsyncTask<JobParameters, Void, String> {

        protected JobParameters mJobParam;

        @Override
        protected String doInBackground(JobParameters... params) {
            // cache system provided job requirements
            mJobParam = params[0];
            try {
                InputStream is = null;
                // Only display the first 50 characters of the retrieved web page content.
                int len = 50;

                URL url = new URL("https://www.google.com");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setReadTimeout(10000); //10sec
                conn.setConnectTimeout(15000); //15sec
                conn.setRequestMethod("GET");
                //Starts the query
                conn.connect();
                int response = conn.getResponseCode();
                Log.d(LOG_TAG, "The response is: " + response);
                is = conn.getInputStream();

                // Convert the input stream to a string
                Reader reader = null;
                reader = new InputStreamReader(is, "UTF-8");
                char[] buffer = new char[len];
                reader.read(buffer);
                return new String(buffer);

            } catch (IOException e) {
                return "Unable to retrieve web page.";
            }
        }

        @Override
        protected void onPostExecute(String result) {
            jobFinished(mJobParam, false);
            Log.i(LOG_TAG, result);
        }
    }
}
```
参考链接：http://www.jianshu.com/p/09d878e4c6ab      
