遇到的问题:当调用getactionbar().setDisplayHomeAsUpEnabled(true);报空指针异常；
解决方法:getSupportActionBar().setDisplayHomeAsUpEnabled(true);

添加和移除Action Bar

    ActionBar的添加非常简单，只需要在AndroidManifest.xml中指定Application或Activity的theme是Theme.Holo或其子类就可以了，而使用Eclipse创建的项目自动就会将Application的theme指定成Theme.Holo，所以ActionBar默认都是显示出来的

而如果想要移除ActionBar的话通常有两种方式，一是将theme指定成Theme.Holo.NoActionBar，表示使用一个不包含ActionBar的主题，二是在Activity中调用以下方法：

ActionBar actionBar = getActionBar();  
actionBar.hide();  

修改Action Bar的图标和标题

默认情况下，系统会使用<application>或者<activity>中icon属性指定的图片来作为ActionBar的图标，但是我们也可以改变这一默认行为。如果我们想要使用另外一张图片来作为ActionBar的图标，可以在<application>或者<activity>中通过logo属性来进行指定。

标题中的内容：
使用label属性来指定一个字符串就可以了

添加Action按钮：
ActionBar还可以根据应用程序当前的功能来提供与其相关的Action按钮，这些按钮都会以图标或文字的形式直接显示在ActionBar上。当然，如果按钮过多，ActionBar上显示不完，多出的一些按钮可以隐藏在overflow里面（最右边的三个点就是overflow按钮），点击一下overflow按钮就可以看到全部的Action按钮了。

当Activity启动的时候，系统会调用Activity的onCreateOptionsMenu()方法来取出所有的Action按钮，我们只需要在这个方法中去加载一个menu资源，并把所有的Action按钮都定义在资源文件里面就可以了。

资源文件如下：

<menu xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    tools:context="com.example.actionbartest.MainActivity" >  
  
    <item  
        android:id="@+id/action_compose"  
        android:icon="@drawable/ic_action_compose"  
        android:showAsAction="always"  
        android:title="@string/action_compose"/>  
    <item  
        android:id="@+id/action_delete"  
        android:icon="@drawable/ic_action_delete"  
        android:showAsAction="always"  
        android:title="@string/action_delete"/>  
    <item  
        android:id="@+id/action_settings"  
        android:icon="@drawable/ic_launcher"  
        android:showAsAction="never"  
        android:title="@string/action_settings"/>  
  
</menu>  

showAsAction则指定了该按钮显示的位置，主要有以下几种值可选：always表示永远显示在ActionBar中，如果屏幕空间不够则无法显示，ifRoom表示屏幕空间够的情况下显示在ActionBar中，不够的话就显示在overflow中，never则表示永远显示在overflow中。

通过Action Bar图标进行导航：
启用ActionBar图标导航的功能，可以允许用户根据当前应用的位置来在不同界面之间切换。比如，A界面展示了一个列表，点击某一项之后进入了B界面，这时B界面就应该启用ActionBar图标导航功能，这样就可以回到A界面。

我们可以通过调用setDisplayHomeAsUpEnabled()方法来启用ActionBar图标导航功能，比如

@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setTitle("天气");  
    setContentView(R.layout.activity_main);  
    ActionBar actionBar = getActionBar();  
    actionBar.setDisplayHomeAsUpEnabled(true);  
}  


可以看到，在ActionBar图标的左侧出现了一个向左的箭头，通常情况下这都表示返回的意思，因此最简单的实现就是在它的点击事件里面加入finish()方法就可以了，如下所示：

@Override  
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
    case android.R.id.home:  
        finish();  
        return true;  
    ……  
    }  
}  

这里有个返回上层activity的步骤：
 一：调用setDisplayHomeAsUpEnabled()方法，并传入true。

二：配置parentActivityName属性
<activity  
    android:name="com.example.actionbartest.MainActivity"  
    android:logo="@drawable/weather"  
    android:parentActivityName="com.example.actionbartest.LaunchActivity" >  
</activity>  

第三步则需要对android.R.id.home这个事件进行一些特殊处理，如下所示：

@Override  
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
    case android.R.id.home:  
        Intent upIntent = NavUtils.getParentActivityIntent(this);  
        if (NavUtils.shouldUpRecreateTask(this, upIntent)) {  
            TaskStackBuilder.create(this)  
                    .addNextIntentWithParentStack(upIntent)  
                    .startActivities();  
        } else {  
            upIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);  
            NavUtils.navigateUpTo(this, upIntent);  
        }  
        return true;  
 
    }
}  

添加Action View:

ActionView是一种可以在ActionBar中替换Action按钮的控件，它可以允许用户在不切换界面的情况下通过ActionBar完成一些较为丰富的操作。比如说，你需要完成一个搜索功能，就可以将SeachView这个控件添加到ActionBar中。

为了声明一个ActionView，我们可以在menu资源中通过actionViewClass属性来指定一个控件，例如可以使用如下方式添加SearchView：

<menu xmlns:android="http://schemas.android.com/apk/res/android" >  
  
    <item  
        android:id="@+id/action_search"  
        android:icon="@drawable/ic_action_search"  
        android:actionViewClass="android.widget.SearchView"  
        android:showAsAction="ifRoom|collapseActionView"  
        android:title="@string/action_search" />  
  
</menu>  

注意在showAsAction属性中我们还声明了一个collapseActionView，这个值表示该控件可以被合并成一个Action按钮。