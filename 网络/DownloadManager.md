# DownlodaManager学习笔记
** 代码见Pracdemo/DownLoad包下代码   也可见Utils包下的DownLoadUtils文件**
## 概念
**DownloadManager:**  从Android 2.3（API level 9）开始Android用系统服务(Service)的方式提供了Download Manager来优化处理长时间的下载操作。Download Manager处理HTTP连接并监控连接中的状态变化以及系统重启来确保每一个下载任务顺利完成。在大多数涉及到下载的情况中使用Download Manager都是不错的选择，特别是当用户切换不同的应用以后下载需要在后台继续进行，以及当下载任务顺利完成非常重要的情况（DownloadManager对于断点续传功能支持很好）。要想使用Download Manager，使用getSystemService方法请求系统的DOWNLOAD_SERVICE服务.

## 第一步：获取服务
```java
String serviceString = Context.DOWNLOAD_SERVICE;  
DownloadManager downloadManager;  
downloadManager = (DownloadManager) getSystemService(serviceString);
```

## 第二步:下载文件
```java
String url="https://raw.githubusercontent.com/815065046/Android-Note/master/%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE/Android%E6%8A%80%E8%83%BD%E6%A0%91.xmind";

        DownloadManager.Request request=new DownloadManager.Request(Uri.parse(url));  //构建一个request
        request.setDestinationInExternalPublicDir("files","my.apk");  //路径和文件名
        request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);

        long id=downloadManager.enqueue(request);  //返回任务ID.
        ```
在这里返回的id变量是系统为当前的下载请求分配的一个唯一的ID，我们可以通过这个ID重新获得这个下载任务，进行一些自己想要进行的操作或者查询下载的状态以及取消下载等等。我们可以通过addRequestHeader方法为DownloadManager.Request对象request添加HTTP头，也可以通过setMimeType方法重写从服务器返回的mime type。我们还可以指定在什么连接状态下执行下载操作。setAllowedNetworkTypes方法可以用来限定在WiFi还是手机网络下进行下载，setAllowedOverRoaming方法可以用来阻止手机在漫游状态下下载。总的来说，就是可配置的
**下面的代码片段用于指定一个较大的文件只能在WiFi下进行下载：**

`request.setAllowedNetworkTypes(Request.NETWORK_WIFI);`
getRecommendedMaxBytesOverMobile类方法（静态方法），返回一个当前手机网络连接下的最大建议字节数，可以来判断下载是否应该限定在WiFi条件下。
调用enqueue方法之后，只要数据连接可用并且Download Manager可用，下载就会开始。
要在下载完成的时候获得一个系统通知（notification）,注册一个广播接受者来接收ACTION_DOWNLOAD_COMPLETE广播，这个广播会包含一个
EXTRA_DOWNLOAD_ID信息在intent中包含了已经完成的这个下载的ID,代码片段如下所示：

```java
IntentFilter filter = new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE);  

BroadcastReceiver receiver = new BroadcastReceiver() {  
  @Override  
  public void onReceive(Context context, Intent intent) {  
    long reference = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);  
    if (myDownloadReference == reference) {  

    }  
  }  
};  
registerReceiver(receiver, filter);


//通知栏样式：
request.setTitle(“File”);  
request.setDescription(“Earthquake XML”);  
```
setNotificationVisibility方法可以用来控制什么时候显示Notification，甚至是隐藏该request的Notification。有以下几个参数：
Request.VISIBILITY_VISIBLE：在下载进行的过程中，通知栏中会一直显示该下载的Notification，当下载完成时，该Notification会被移除，这是默认的参数值。
Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED：在下载过程中通知栏会一直显示该下载的Notification，在下载完成后该Notification会继续显示，直到用户点击该Notification或者消除该Notification。
Request.VISIBILITY_VISIBLE_NOTIFY_ONLY_COMPLETION：只有在下载完成后该Notification才会被显示。
Request.VISIBILITY_HIDDEN：不显示该下载请求的Notification。
默认情况下，所有通过Download Manager下载的文件都保存在一个共享下载缓存中，使用系统生成的文件名每一个Request对象都可以制定一个下载保存的地址，通常情况下，所有的下载文件都应该保存在外部存储中.

在默认的情况下，通过Download Manager下载的文件是不能被Media Scanner扫描到的，进而这些下载的文件（音乐、视频等）就不会在Gallery和
Music Player这样的应用中看到。
为了让下载的音乐文件可以被其他应用扫描到，我们需要调用Request对象的allowScaningByMediaScanner方法。

取消下载可以使用remove方法。

查询：

你可以通过查询Download Manager来获得下载任务的状态，进度，以及各种细节，通过query方法返回一个包含了下载任务细节的Cursor。
query方法传递一个DownloadManager.Query对象作为参数，通过DownloadManager.Query对象的setFilterById方法可以筛选我们希望查询的下
载任务的ID。也可以使用setFilterByStatus方法筛选我们希望查询的某一种状态的下载任务，传递的参数是DownloadManager.STATUS_*常量，可以指定
正在进行、暂停、失败、完成四种状态。
Download Manager包含了一系列COLUMN_*静态String常量，可以用来查询Cursor中的结果列索引。我们可以查询到下载任务的各种细节，包括状态，
文件大小，已经下载的字节数，标题，描述，URI，本地文件名和URI，媒体类型以及Media Provider download URI。
下面的代码段是通过注册监听下载完成事件的广播接受者来查询下载完成文件的本地文件名和URI的实现方法：
```java

public void onReceive(Context context, Intent intent) {  
  long reference = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);  
  if (myDownloadReference == reference) {  

    Query myDownloadQuery = new Query();  
    myDownloadQuery.setFilterById(reference);  

    Cursor myDownload = downloadManager.query(myDownloadQuery);  
    if (myDownload.moveToFirst()) {  
      int fileNameIdx =   
        myDownload.getColumnIndex(DownloadManager.COLUMN_LOCAL_FILENAME);  
      int fileUriIdx =   
        myDownload.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI);  

      String fileName = myDownload.getString(fileNameIdx);  
      String fileUri = myDownload.getString(fileUriIdx);  

      // TODO Do something with the file.  
      Log.d(TAG, fileName + " : " + fileUri);  
    }  
    myDownload.close();  

  }  
}  
```
 对于暂停和失败的下载，我们可以通过查询COLUMN_REASON列查询出原因的整数码。
对于STATUS_PAUSED状态的下载，可以通过DownloadManager.PAUSED_*静态常量来翻译出原因的整数码，进而判断出下载是由于等待网络连接
还是等待WiFi连接还是准备重新下载三种原因而暂停。
对于STATUS_FAILED状态的下载，我们可以通过DownloadManager.ERROR_*来判断失败的原因，可能是错误码（失败原因）包括没有存储设备，存储空间不足，重复的文件名，或者HTTP errors。
下面的代码是如何查询出当前所有的暂停的下载任务，提取出暂停的原因以及文件名称，下载标题以及当前进度的实现方法：
```java
// Obtain the Download Manager Service.  
String serviceString = Context.DOWNLOAD_SERVICE;  
DownloadManager downloadManager;  
downloadManager = (DownloadManager)getSystemService(serviceString);  

// Create a query for paused downloads.  
Query pausedDownloadQuery = new Query();  
pausedDownloadQuery.setFilterByStatus(DownloadManager.STATUS_PAUSED);  

// Query the Download Manager for paused downloads.  
Cursor pausedDownloads = downloadManager.query(pausedDownloadQuery);  

// Find the column indexes for the data we require.  
int reasonIdx = pausedDownloads.getColumnIndex(DownloadManager.COLUMN_REASON);  
int titleIdx = pausedDownloads.getColumnIndex(DownloadManager.COLUMN_TITLE);  
int fileSizeIdx =   
  pausedDownloads.getColumnIndex(DownloadManager.COLUMN_TOTAL_SIZE_BYTES);      
int bytesDLIdx =   
  pausedDownloads.getColumnIndex(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR);  

// Iterate over the result Cursor.  
while (pausedDownloads.moveToNext()) {  
  // Extract the data we require from the Cursor.  
  String title = pausedDownloads.getString(titleIdx);  
  int fileSize = pausedDownloads.getInt(fileSizeIdx);  
  int bytesDL = pausedDownloads.getInt(bytesDLIdx);  

  // Translate the pause reason to friendly text.  
  int reason = pausedDownloads.getInt(reasonIdx);  
  String reasonString = "Unknown";  
  switch (reason) {  
    case DownloadManager.PAUSED_QUEUED_FOR_WIFI :   
      reasonString = "Waiting for WiFi"; break;  
    case DownloadManager.PAUSED_WAITING_FOR_NETWORK :   
      reasonString = "Waiting for connectivity"; break;  
    case DownloadManager.PAUSED_WAITING_TO_RETRY :  
      reasonString = "Waiting to retry"; break;  
    default : break;  
  }  

  // Construct a status summary  
  StringBuilder sb = new StringBuilder();  
  sb.append(title).append("\n");  
  sb.append(reasonString).append("\n");  
  sb.append("Downloaded ").append(bytesDL).append(" / " ).append(fileSize);  

  // Display the status   
  Log.d("DOWNLOAD", sb.toString());  
}  

// Close the result Cursor.  
pausedDownloads.close();  
```

参考：http://blog.csdn.net/carrey1989/article/details/8060155
