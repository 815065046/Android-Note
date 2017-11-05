代码见Content包下: 

观察特定Uri的步骤如下：
 
     1、    创建我们特定的ContentObserver派生类，必须重载父类构造方法，必须重载onChange()方法去处理回调后的功能实现
     2、    利用context.getContentResolover()获得ContentResolove对象，接着调用registerContentObserver()方法去注册内容观察者
     3、    由于ContentObserver的生命周期不同步于Activity和Service等，因此，在不需要时，需要手动的调用
             unregisterContentObserver()去取消注册

注意：每个ContentProvider数据源发生改变后，如果想通知其监听对象， 例如ContentObserver时，必须在其对应方法 update / 
  insert / delete时，显示的调用this.getContentReslover（）.notifychange(uri , null)方法，回调监听处理逻辑。否则，我们
  的ContentObserver是不会监听到数据发生改变的。

具体见代码MyContentProvider代码：

package com.lans.lwk.pracdemo.Content;

import android.content.ContentProvider;
import android.content.ContentValues;
import android.content.UriMatcher;
import android.database.Cursor;
import android.net.Uri;
import android.util.Log;

public class MyContentProvider extends ContentProvider {
    static UriMatcher matcher;
    static{
        matcher=new UriMatcher(UriMatcher.NO_MATCH);
        matcher.addURI("com.android.cc","/*",2);
    }
    public MyContentProvider() {
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // Implement this to handle requests to delete one or more rows.
        Log.i("hello","delete");

        return 0;
    }

    @Override
    public String getType(Uri uri) {
        // TODO: Implement this to handle requests for the MIME type of the data
        // at the given URI.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // TODO: Implement this to handle requests to insert a new row.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public boolean onCreate() {
        // TODO: Implement this to initialize your content provider on startup.
        return false;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        // TODO: Implement this to handle query requests from clients.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        // TODO: Implement this to handle requests to update one or more rows.
        throw new UnsupportedOperationException("Not yet implemented");
    }
}

ContentObsver 代码：
package com.lans.lwk.pracdemo.Content;


import android.content.Context;
import android.database.ContentObserver;
import android.database.Cursor;
import android.net.Uri;
import android.os.Handler;
import android.provider.Settings;
import android.util.Log;

/**
 * Created by Li on 2017/10/29.
 * ContentObserver测试程序监控程序飞行模式的开关
 */

public class ContentObsver extends ContentObserver {
    private Context context;
    private Handler handler;
    /**
     * Creates a content observer.
     *
     * @param handler The handler to run {@link #onChange} on, or null if none.
     */
    public ContentObsver(Context context, Handler handler) {
        super(handler);
        this.handler=handler;
        this.context=context;
    }

    @Override
    public void onChange(boolean selfChange) {
             Log.i("hello","onChange");

   /*     // 系统是否处于飞行模式下
        try {
            int isAirplaneOpen = Settings.System.getInt(context.getContentResolver(), Settings.System.AIRPLANE_MODE_ON);
            Log.i("TAG", " isAirplaneOpen -----> " +isAirplaneOpen) ;
            handler.obtainMessage(2,isAirplaneOpen).sendToTarget() ;
        }
        catch (Settings.SettingNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }*/
        handler.obtainMessage(2,2).sendToTarget() ;
        super.onChange(selfChange);

    }
}

