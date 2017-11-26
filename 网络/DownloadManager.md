# DownlodaManagerѧϰ�ʼ�
** �����Pracdemo/DownLoad���´���   Ҳ�ɼ�Utils���µ�DownLoadUtils�ļ�**
## ����
**DownloadManager:**  ��Android 2.3��API level 9����ʼAndroid��ϵͳ����(Service)�ķ�ʽ�ṩ��Download Manager���Ż�����ʱ������ز�����Download Manager����HTTP���Ӳ���������е�״̬�仯�Լ�ϵͳ������ȷ��ÿһ����������˳����ɡ��ڴ�����漰�����ص������ʹ��Download Manager���ǲ����ѡ���ر��ǵ��û��л���ͬ��Ӧ���Ժ�������Ҫ�ں�̨�������У��Լ�����������˳����ɷǳ���Ҫ�������DownloadManager���ڶϵ���������֧�ֺܺã���Ҫ��ʹ��Download Manager��ʹ��getSystemService��������ϵͳ��DOWNLOAD_SERVICE����.

## ��һ������ȡ����
```java
String serviceString = Context.DOWNLOAD_SERVICE;  
DownloadManager downloadManager;  
downloadManager = (DownloadManager) getSystemService(serviceString);
```

## �ڶ���:�����ļ�
```java
String url="https://raw.githubusercontent.com/815065046/Android-Note/master/%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE/Android%E6%8A%80%E8%83%BD%E6%A0%91.xmind";

        DownloadManager.Request request=new DownloadManager.Request(Uri.parse(url));  //����һ��request
        request.setDestinationInExternalPublicDir("files","my.apk");  //·�����ļ���
        request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);

        long id=downloadManager.enqueue(request);  //��������ID.
        ```
�����ﷵ�ص�id������ϵͳΪ��ǰ��������������һ��Ψһ��ID�����ǿ���ͨ�����ID���»������������񣬽���һЩ�Լ���Ҫ���еĲ������߲�ѯ���ص�״̬�Լ�ȡ�����صȵȡ����ǿ���ͨ��addRequestHeader����ΪDownloadManager.Request����request���HTTPͷ��Ҳ����ͨ��setMimeType������д�ӷ��������ص�mime type�����ǻ�����ָ����ʲô����״̬��ִ�����ز�����setAllowedNetworkTypes�������������޶���WiFi�����ֻ������½������أ�setAllowedOverRoaming��������������ֹ�ֻ�������״̬�����ء��ܵ���˵�����ǿ����õ�
**����Ĵ���Ƭ������ָ��һ���ϴ���ļ�ֻ����WiFi�½������أ�**

`request.setAllowedNetworkTypes(Request.NETWORK_WIFI);`
getRecommendedMaxBytesOverMobile�෽������̬������������һ����ǰ�ֻ����������µ�������ֽ������������ж������Ƿ�Ӧ���޶���WiFi�����¡�
����enqueue����֮��ֻҪ�������ӿ��ò���Download Manager���ã����ؾͻῪʼ��
Ҫ��������ɵ�ʱ����һ��ϵͳ֪ͨ��notification��,ע��һ���㲥������������ACTION_DOWNLOAD_COMPLETE�㲥������㲥�����һ��
EXTRA_DOWNLOAD_ID��Ϣ��intent�а������Ѿ���ɵ�������ص�ID,����Ƭ��������ʾ��

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


//֪ͨ����ʽ��
request.setTitle(��File��);  
request.setDescription(��Earthquake XML��);  
```
setNotificationVisibility����������������ʲôʱ����ʾNotification�����������ظ�request��Notification�������¼���������
Request.VISIBILITY_VISIBLE�������ؽ��еĹ����У�֪ͨ���л�һֱ��ʾ�����ص�Notification�����������ʱ����Notification�ᱻ�Ƴ�������Ĭ�ϵĲ���ֵ��
Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED�������ع�����֪ͨ����һֱ��ʾ�����ص�Notification����������ɺ��Notification�������ʾ��ֱ���û������Notification����������Notification��
Request.VISIBILITY_VISIBLE_NOTIFY_ONLY_COMPLETION��ֻ����������ɺ��Notification�Żᱻ��ʾ��
Request.VISIBILITY_HIDDEN������ʾ�����������Notification��
Ĭ������£�����ͨ��Download Manager���ص��ļ���������һ���������ػ����У�ʹ��ϵͳ���ɵ��ļ���ÿһ��Request���󶼿����ƶ�һ�����ر���ĵ�ַ��ͨ������£����е������ļ���Ӧ�ñ������ⲿ�洢��.

��Ĭ�ϵ�����£�ͨ��Download Manager���ص��ļ��ǲ��ܱ�Media Scannerɨ�赽�ģ�������Щ���ص��ļ������֡���Ƶ�ȣ��Ͳ�����Gallery��
Music Player������Ӧ���п�����
Ϊ�������ص������ļ����Ա�����Ӧ��ɨ�赽��������Ҫ����Request�����allowScaningByMediaScanner������

ȡ�����ؿ���ʹ��remove������

��ѯ��

�����ͨ����ѯDownload Manager��������������״̬�����ȣ��Լ�����ϸ�ڣ�ͨ��query��������һ����������������ϸ�ڵ�Cursor��
query��������һ��DownloadManager.Query������Ϊ������ͨ��DownloadManager.Query�����setFilterById��������ɸѡ����ϣ����ѯ����
�������ID��Ҳ����ʹ��setFilterByStatus����ɸѡ����ϣ����ѯ��ĳһ��״̬���������񣬴��ݵĲ�����DownloadManager.STATUS_*����������ָ��
���ڽ��С���ͣ��ʧ�ܡ��������״̬��
Download Manager������һϵ��COLUMN_*��̬String����������������ѯCursor�еĽ�������������ǿ��Բ�ѯ����������ĸ���ϸ�ڣ�����״̬��
�ļ���С���Ѿ����ص��ֽ��������⣬������URI�������ļ�����URI��ý�������Լ�Media Provider download URI��
����Ĵ������ͨ��ע�������������¼��Ĺ㲥����������ѯ��������ļ��ı����ļ�����URI��ʵ�ַ�����
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
 ������ͣ��ʧ�ܵ����أ����ǿ���ͨ����ѯCOLUMN_REASON�в�ѯ��ԭ��������롣
����STATUS_PAUSED״̬�����أ�����ͨ��DownloadManager.PAUSED_*��̬�����������ԭ��������룬�����жϳ����������ڵȴ���������
���ǵȴ�WiFi���ӻ���׼��������������ԭ�����ͣ��
����STATUS_FAILED״̬�����أ����ǿ���ͨ��DownloadManager.ERROR_*���ж�ʧ�ܵ�ԭ�򣬿����Ǵ����루ʧ��ԭ�򣩰���û�д洢�豸���洢�ռ䲻�㣬�ظ����ļ���������HTTP errors��
����Ĵ�������β�ѯ����ǰ���е���ͣ������������ȡ����ͣ��ԭ���Լ��ļ����ƣ����ر����Լ���ǰ���ȵ�ʵ�ַ�����
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

�ο���http://blog.csdn.net/carrey1989/article/details/8060155
