一，结构图
<?xmlversion="1.0"encoding="utf-8"?>
<manifest>

    <uses-sdk/> 
    <uses-configuration/> 
    <uses-feature/>  

    <uses-permission/>
    <permission/>
    <permission-tree/>
    <permission-group/>
    <instrumentation/> 

    <supports-screens/>

    <application> 
       <activity> 
           <intent-filter>
               <action/> 
               <category/> 
           </intent-filter> 
      </activity>

       <activity-alias> 
           <intent-filter></intent-filter> 
           <meta-data/> 
      </activity-alias> 

       <service> 
           <intent-filter></intent-filter> 
           <meta-data/> 
       </service>

       <receiver>
           <intent-filter></intent-filter> 
           <meta-data/> 
       </receiver> 

       <provider> 
           <grant-uri-permission/>
           <meta-data/> 
       </provider> 

       <uses-library/> 
    </application>  

</manifest>

Manifest:属性
<manifest  xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.finddreams.csdn"
          android:sharedUserId="string"
          android:sharedUserLabel="string resource"
          android:versionCode="integer"
          android:versionName="string"
          android:installLocation=["auto" | "internalOnly" | "preferExternal"] >
</manifest>

sharedUserId
表明数据权限，因为默认情况下，Android给每个APK分配一个唯一的UserID，所以是默认禁止不同APK访问共享数据的。若要共享数据，第一可以采用Share Preference方法，第二种就可以采用sharedUserId了，将不同APK的sharedUserId都设为一样，则这些APK之间就可以互相共享数据了。

installLocation
安装参数，是Android2.2中的一个新特性，installLocation有三个值可以选择：internalOnly、auto、preferExternal
选择preferExternal,系统会优先考虑将APK安装到SD卡上(当然最终用户可以选择为内部ROM存储上，如果SD存储已满，也会安装到内部存储上)
选择auto，系统将会根据存储空间自己去适应
选择internalOnly是指必须安装到内部才能运行

Application:属性
一个AndroidManifest.xml中必须含有一个Application标签，这个标签声明了每一个应用程序的组件及其属性(如icon,label,permission等)
<application  android:allowClearUserData=["true" | "false"]
             android:allowTaskReparenting=["true" | "false"]
             android:backupAgent="string"
             android:debuggable=["true" | "false"]
             android:description="string resource"
             android:enabled=["true" | "false"]
             android:hasCode=["true" | "false"]
             android:icon="drawable resource"
             android:killAfterRestore=["true" | "false"]
             android:label="string resource"
             android:manageSpaceActivity="string"
             android:name="string"
             android:permission="string"
             android:persistent=["true" | "false"]
             android:process="string"
             android:restoreAnyVersion=["true" | "false"]
             android:taskAffinity="string"
             android:theme="resource or theme" >
</application>

android:restoreAnyVersion
用来表明应用是否准备尝试恢复所有的备份，甚至该备份是比当前设备上更要新的版本，默认是false；

3、Activity:属性

<activity android:allowTaskReparenting=["true" | "false"]
          android:alwaysRetainTaskState=["true" | "false"]
          android:clearTaskOnLaunch=["true" | "false"]
          android:configChanges=["mcc", "mnc", "locale",
                                 "touchscreen", "keyboard", "keyboardHidden",
                                 "navigation", "orientation", "screenLayout",
                                 "fontScale", "uiMode"]
          android:enabled=["true" | "false"]
          android:excludeFromRecents=["true" | "false"]
          android:exported=["true" | "false"]
          android:finishOnTaskLaunch=["true" | "false"]
          android:icon="drawable resource"
          android:label="string resource"
          android:launchMode=["multiple" | "singleTop" |
                              "singleTask" | "singleInstance"]
          android:multiprocess=["true" | "false"]
          android:name="string"
          android:noHistory=["true" | "false"]  
          android:permission="string"
          android:process="string"
          android:screenOrientation=["unspecified" | "user" | "behind" |
                                     "landscape" | "portrait" |
                                     "sensor" | "nosensor"]
          android:stateNotNeeded=["true" | "false"]
          android:taskAffinity="string"
          android:theme="resource or theme"
          android:windowSoftInputMode=["stateUnspecified",
                                       "stateUnchanged", "stateHidden",
                                       "stateAlwaysHidden", "stateVisible",
                                       "stateAlwaysVisible", "adjustUnspecified",
                                       "adjustResize", "adjustPan"] >   
</activity>
1、android:alwaysRetainTaskState
是否保留状态不变， 比如切换回home, 再从新打开，activity处于最后的状态。比如一个浏览器拥有很多状态(当打开了多个TAB的时候)，用户并不希望丢失这些状态时，此时可将此属性设置为true

2、android:excludeFromRecents
是否可被显示在最近打开的activity列表里，默认是false

3、android:multiprocess
是否允许多进程，默认是false

4、android:screenOrientation
activity显示的模式
默认为unspecified：由系统自动判断显示方向
landscape横屏模式，宽度比高度大
portrait竖屏模式, 高度比宽度大
user模式，用户当前首选的方向
behind模式：和该Activity下面的那个Activity的方向一致(在Activity堆栈中的)
sensor模式：有物理的感应器来决定。如果用户旋转设备这屏幕会横竖屏切换
nosensor模式：忽略物理感应器，这样就不会随着用户旋转设备而更改了

5、android:alwaysRetainTaskState
是否保留状态不变，比如切换回home, 再从新打开，activity处于最后的状态。比如一个浏览器拥有很多状态(当打开了多个TAB的时候)，用户并不希望丢失这些状态时，此时可将此属性设置为true