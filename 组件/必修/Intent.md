  一、什么是Intent,有什么作用?
    Android的应用程序包括四大组件：Activity、contentProvider、Service、BroadcastReceiver,为了方便不同组件之间的交流通信，应用程序就采用了一种统一的方式启动组件及传递数据，即使用Intent。            
    Intent封装了Android应用程序需要启动某个组件的"意图"，Intent类的对象是组件间的通信载体，一个Intent对象就是一组信息，其包含接收Intent组件所关心的信息（如action 和 Data）和Android 系统关心的信息(如Category等)。也就是说，发送"意图"的组件通过Intent对象所包含的内容，来启动指定的(即Component属性)或通过筛选(即Action&Category属性)的某(些)组件，然后实施相应的动作（即Action属性）并传递相应的数据(即Data属性)以便完成相应的动作。

二、Intent是如何实现组件间相互调用？
首先，发出"意图"的组件，通过调用Context.startActivity(intent)开始启动组件：发出"意图"的组件传入已经构好的Intent对象(为一组信息，它决定了是否能够成功的启动另一个组件）；
   然后，调用"动作"实际的执行着Instrumentation对象，它是整个应用激活的Activity管理者，集中负责应用内所有Activity的运行。它有一个隐藏的方法execStartActivity方法，就是负责根据Intent启动Activity。该方法去掉一些细节，它做得最重要的事情，就是将此调用，通过RPC的方式，传递到ActivityManagerService。
   最后，ActivityManagerService会将这些关键信息递交给另一个服务PackageManagerService，此服务掌握整个软件包及其各组件的信息，它会将传递过来的Intent，与已知的所有Intent Filters进行匹配（如果带有Component信息，就不用比了）。如果找到了，就把相关Component的信息通知回AcitivityManagerService，在这里，会完成启动Activity这个很多细节的事情。

三、属性
1.Action属性
 Action属性为一个普通的字符串，它代表了该Intent对象要完成一个什么样的"动作"。当我们为Intent对象指明了一个action时，被启动的组件就会依照这个动作的指示表现出相应的行为，比如查看、拨打、编辑等，需要注意的是一个组件(如Activity)只能有一个action。我们可以方便自己的应用程序组件之间的通信，自定义action的(字符串)创建新的动作；也可以直接使用Intent类中的静态成员变量，比如ACTION_VIEW，ACTION_PICK，它们是Android中为action属性预定义的一批action变量。

2.Data属性
   Action属性为Intent对象描述了一个"动作"，那么Data属性就为Intent对象的Action属性提供了操作的数据。这里需要注意的是，Data属性只接受一个Uri对象，一个Uri对象通常通过如下形式的字符串来表示：
Uri字符串格式：scheme://host:port/path 举例： content://com.android.contacts/contacts/1或tel://18819463209

其中包含:mimeType 类型 scheme 域名 host port path

3.Catagory属性
    通过Action，配合Data或Type属性可以准确的表达出一个完整的意图了。但为了使的"意图"更加精确，我们也给意图添加一些约束，这个约束由"意图"的Catagory属性实现。一个意图只能指定一个action属性，但是可以添加一个或多个Catagory属性。Category属性可以自定义字符串实现，但为了方便不同应用之间的通信还可以设置系统预定义的Category常量。调用方法addCategory 用来为Intent 添加一个Category，方法removeCategory 用来移除一个Category；getCategories方法返回已定义的Category。
4.Flags属性
   能识别，有输入，整个Intent基本就完整了，但还有一些附件的指令，需要放在Flags中带过去。顾名思义，Flags是一个整形数，有一些列的标志 位构成，这些标志，是用来指明运行模式的。比如，你期望这个意图的执行者，和你运行在两个完全不同的任务中（或说进程也无妨吧...），就需要设置FLAG_ACTIVITY_NEW_TASK 的标志位。

四、URI（统一资源标识符）
Uri字符串格式：scheme://host:port/path




