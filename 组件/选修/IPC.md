IPC方式：

1.使用Bundle
四大组件的三大组件Activity,Service。Receiver都支持在Intent中传递Bundle数据，Bindle实现了Parcelable接口，所以可以传输。
2.使用文件共享
在Android中可以进行并发的读/写，但是存在并发读写问一体，一般要求数据的同步要求不高。
Sharedperferences不能进行文件共享的方式，因为在内存中有它的缓存。它通过键值对的方式来存储数据，底层是xml。
3.Messager
Messager实现IPC通信，底层也是使用了AIDL方式。和AIDL方式不同的是，Messager方式是利用Handler形式处理，因此，它是线程安全的，这也表示它不支持并发处理；而AIDL方式是非线程安全的，支持并发处理，因此，我们使用AIDL方式时需要保证代码的线程安全。
自定义的parcelable字段不能通过object传输。
Client端代码：
privateMessengerreplymessenger=newMessenger(newHandler(){
@Override
publicvoidhandleMessage(Messagemsg){
if(msg.what==5){
Log.i(TAG,"handleMessage:"+msg.getData().getString("msg"));
}


super.handleMessage(msg);
}
});

privateServiceConnectionconnection=newServiceConnection(){
@Override
Publicvoid   onServiceConnected(ComponentNamename,IBinderservice){
Messenger = new Messenger(service);
Messagemessage=Message.obtain(null,MyService2.GET_RESULT);
Bundlebundle=newBundle();
bundle.putString("msg","helloservce");
message.replyTo=replymessenger;
message.setData(bundle);
try{
messenger.send(message);
}catch(RemoteExceptione){
e.printStackTrace();
}


}

@Override
publicvoidonServiceDisconnected(ComponentNamename){

}
};
@Override
protectedvoidonCreate(BundlesavedInstanceState){
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
Log.i(TAG,"onCreate:");
bindService(newIntent(this,MyService2.class),connection,BIND_AUTO_CREATE);
服务器端代码：
publicstaticfinalintGET_RESULT=1;
privateMessengermessenger=newMessenger(newHandler(){
@Override
publicvoidhandleMessage(Messagemsg){
if(msg.what==GET_RESULT){
try{
Messagemessages=Message.obtain(null,5);
Bundlebundle=newBundle();
bundle.putString("msg","ClientHELLO");
messages.setData(bundle);
msg.replyTo.send(messages);
Log.i(TAG,"handleMessage:"+msg.getData().getString("msg"));
}catch(Exceptione){
e.printStackTrace();
}
}else{
super.handleMessage(msg);
}



}
});
publicMyService2(){
}

@Override
publicvoidonCreate(){
super.onCreate();

}

@Override
publicIBinderonBind(Intentintent){
//TODO:Returnthecommunicationchanneltotheservice.
returnmessenger.getBinder();
}


AIDL:
服务器端代码：
privateCopyOnWriteArrayList<Book>booklist=newCopyOnWriteArrayList<>();
privateIBookManager.Stubstub=newIBookManager.Stub(){
@Override
publicvoidbasicTypes(intanInt,longaLong,booleanaBoolean,floataFloat,doubleaDouble,StringaString)throwsRemoteException{

}

@Override
publicList<Book>getBook()throwsRemoteException{
returnbooklist;
}

@Override
publicvoidaddBook(Bookbook)throwsRemoteException{

booklist.add(book);


}
};
客户端代码：
privateIBookManagerstub;
privateIBinder.DeathRecipientdeathRecipient=newIBinder.DeathRecipient(){
@Override
publicvoidbinderDied(){
if(stub==null)return;
stub.asBinder().unlinkToDeath(deathRecipient,0);
stub=null;
//重新连接

}
};
privateServiceConnectionconnection=newServiceConnection(){
@Override
publicvoidonServiceConnected(ComponentNamename,IBinderservice){
stub=IBookManager.Stub.asInterface(service);
try{
stub.asBinder().linkToDeath(deathRecipient,0);
List<Book>book=stub.getBook();
Log.i(TAG,"onServiceConnected:"+book.toString());
}catch(RemoteExceptione){
e.printStackTrace();
}

}

@Override
publicvoidonServiceDisconnected(ComponentNamename){

}
};


AIDL支持的数据类型：

基本数据类型（int ,long ,char,boolean,double）
String 和CharSequence
ArrayList
HashMap
Parcelable
AIDL本身


AIDL只能声明方法，不能声明静态常量

contentprovider：
Content://auth/path/#/*
*表示匹配任意长度的任意字符
#表示匹配任意长度的数字
"vnd.android.cursor.item/vnd.draw.li.com.provider.table"
"vnd.android.cursor.dir/vnd.draw.li.com.provider.table";
publicstaticfinalStringTAG="MyContentProvider";
//Uriuri=Uri.parse("content://draw.li.com.provider/table");
staticUriMatcheruriMatcher;
static{
uriMatcher=newUriMatcher(UriMatcher.NO_MATCH);
uriMatcher.addURI("draw.li.com.provider","book",1);
}

publicMyContentProvider(){
Log.i(TAG,"delete:"+"HELLOsss");
}

@Override
publicintdelete(Uriuri,Stringselection,String[]selectionArgs){
//Implementthistohandlerequeststodeleteoneormorerows.
intcow=0;
switch(uriMatcher.match(uri)){
case1:
Log.i(TAG,"delete:"+"HELLO");
break;

}
returncow;
}

@Override
publicStringgetType(Uriuri){
//TODO:ImplementthistohandlerequestsfortheMIMEtypeofthedata
//atthegivenURI.
Log.i(TAG,"delete:"+"HELLOsd");
switch(uriMatcher.match(uri)){

case1:
return"vnd.android.cursor.item/vnd.draw.li.com.provider.table";

}
returnnull;
}

@Override
publicUriinsert(Uriuri,ContentValuesvalues){
//TODO:Implementthistohandlerequeststoinsertanewrow.
thrownewUnsupportedOperationException("Notyetimplemented");
}

@Override
publicbooleanonCreate(){
//TODO:Implementthistoinitializeyourcontentprovideronstartup.
returnfalse;
}

@Override
publicCursorquery(Uriuri,String[]projection,Stringselection,
String[]selectionArgs,StringsortOrder){
//TODO:Implementthistohandlequeryrequestsfromclients.
thrownewUnsupportedOperationException("Notyetimplemented");
}

@Override
publicintupdate(Uriuri,ContentValuesvalues,Stringselection,
String[]selectionArgs){
//TODO:Implementthistohandlerequeststoupdateoneormorerows.
thrownewUnsupportedOperationException("Notyetimplemented");
}
客户端：
getContentResolver().delete(Uri.parse("content://draw.li.com.provider/book"),null,null);

<provider
android:name=".MyContentProvider"
android:authorities="draw.li.com.provider"
android:enabled="true"
android:exported="true"
android:readPermission="s"
android:writePermission=""></provider>

http://www.2cto.com/kf/201404/296974.html
contentprovider详解
除了onCreat（）增删改查都运行在Binder线程池中。
主要以表格形式组织数据
Socket通信