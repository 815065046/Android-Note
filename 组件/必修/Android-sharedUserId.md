Android给每个APK进程分配一个单独的用户空间,其manifest中的userid就是对应一个Linux用户
(Android 系统是基于Linux)的.
所以不同APK(用户)间互相访问数据默认是禁止的.
但是它也提供了2种APK间共享数据的形式:
1. Share Preference. / Content Provider
APK可以指定接口和数据给任何其他APK读取. 需要自己实现接口和Share的数据.
本文对于这个不做详细解释

2. Shared User id
通过Shared User id,拥有同一个User id的多个APK可以配置成运行在同一个进程中.所以默认就是
可以互相访问任意数据. 也可以配置成运行成不同的进程, 同时可以访问其他APK的数据目录下的
数据库和文件.就像访问本程序的数据一样.
比如某个公司开发了多个Android 程序, 那么可以把数据,图片等资源集中放到APK  A中去. 然后
这个公司的所有APK都使用同一个User ID, 那么所有的资源都可以从APK A中读取.

举个例子:
APK A 和APK B 都是C公司的产品,那么如果用户从APK A中登陆成功.那么打开APK B的时候就不用
再次登陆. 具体实现就是 A和B设置成同一个User ID:
    * 在2个APK的AndroidManifest.xml 配置User ID:
    <manifest xmlns:android="http://schemas.android.com/apk/res/android" 
    package="com.android.demo.a1"
    android:sharedUserId="com.c">
   这个"com.c" 就是user id, 然后packagename APK A就是上面的内容,  APK B可能
   是"com.android.demo.b1" 这个没有限制

这个设定好之后, APK B就可以像打开本地数据库那样 打开APK A中的数据库了.
APK A把登陆信息存放在A的数据目录下面. APK B每次启动的时候读取APK A下面的数据库
判断是否已经登陆:
APK B中的代码:
            friendContext = this.createPackageContext(
                    "com.android.demo.a1",
                    Context.CONTEXT_IGNORE_SECURITY);
通过A的package name 就可以得到A的 packagecontext
通过这个context就可以直接打开数据库