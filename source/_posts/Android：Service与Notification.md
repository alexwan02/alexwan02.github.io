title: Android：Notification基础
date: 2016-07-22 14:48:43
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1467529028/android/8ff23095a2a4e04af26ca63642bfdea3_b.png
categories: android-aosp
tags: android-aosp
---
## Notification 通知
所谓通知就是在通知栏或状态显示的消息UI。在app中通知一般用在即时通讯、音乐播放、闹钟等功能上。因此了解如何创建通知并且在多种不同配置的设备上进行适配，还有很有必要的。
## 一、创建通知
从Android 3.0版本开始，Android 添加了Notification.Builder相关API来帮助实现通知功能，但是对版本3.0以下的Android通知，需要通过v4包中的[NotificationCompat.Builder](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)类来创建通知，也是本文主要的创建通知的方式。调用[NotificationCompat.Builder.build()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#build()创建[Notification](https://developer.android.com/reference/android/app/Notification.html)对象，然后通过[NotificationManager.notify()](https://developer.android.com/reference/android/app/NotificationManager.html#notify(int, android.app.Notification)来发送通知

### 1. 通知的必要元素
通知对象必须要指定至少以下几个元素
- 小图标，通过调用[setSmallIcon()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setSmallIcon(int)
- 标题，通过调用[setContentTitle()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentTitle(java.lang.CharSequence)
- 通知文本，通过调用[setContentText()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentText(java.lang.CharSequence)

### 2. 其他可选的通知设置
通知除了三个必要的元素，其他的内容设置都是可选项，可参考[NotificationCompat.Builder](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)查看相关API
1. ）通知行为
尽管这个是可选的，但在实际开发的过程中至少要为通知添加一个指定的行为。比如：点击通知时跳转到应用中指定的Activity等。
通知可以提供多个Action，用来触发用户点击通知时的行为，常用方式就是跳转到应用指定页面。或者推迟闹钟、快速回复短信（需要Android 4.1及以上版本）等。在添加这些功能时，要保证应用已经实现了相应的功能。
在创建通知时，通过[PendingIntent](https://developer.android.com/reference/android/app/PendingIntent.html)为通知指定点击时的要启动的Activity、Service或BroadcastReceiver。如下面的示例代码中，调用[NotificationCompat.Builder](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)的[setContentIntent()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))方法设置启动Activity的PendingIntent对象，来实现点击通知跳转到指定Activity的功能
	```java
	...
	NotificationCompat.Builder mBuilder = 
	    new NotificationCompat.Builder(this)
	        .setSmallIcon(R.drawable.notification_icon)
	        .setContentTitle("My Notification")
	        .setContexttext("Hello World");
	...
	// 创建启动的Activity的Intent
	Intent notifyIntent = new Intent(this , ResultActivity.class);
	// 设置Acitivity的新的空栈
	notifyIntent.setFlag(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
	// 创建PendingIntent
	PendingIntent notifyPendingIntent = 
	    PendingIntent.getActivity(this , 0 , notifyIntent , PendingIntent.FLAG_UPDATE_CURRENT);
	// 设置通知的ContentIntent
	builder.setContentIntent(notifyPendingIntent);
	// 发送通知
	...
	```
2. ）通知优先级
根据定义的优先级可以决定通知在通知栏中的显示位置。设置通知优先级，调用[NotificationCompat.Builder.setPriority()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setPriority(int))方法，参数为NotificationCompat中[PRIORITY_MIN（-2）](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html#PRIORITY_MIN)到[PRIORITY_MAX（2）](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html#PRIORITY_MAX)之前的值。默认为PRIORITY_DEFAULT（0）；查看[Correctly set and manage notification priority](https://developer.android.com/design/patterns/notifications.html)，为通知设置合适的优先级。

3. ）创建简单的通知
点击通知时打开指定Actviity。
	```java
	NotificationCompat.Builder mBuilder =
	        new NotificationCompat.Builder(this)
	        .setSmallIcon(R.drawable.notification_icon)
	        .setContentTitle("My notification")
	        .setContentText("Hello World!");
	// Creates an explicit intent for an Activity in your app
	Intent resultIntent = new Intent(this, ResultActivity.class);
	
	// The stack builder object will contain an artificial back stack for the
	// started Activity.
	// TaskStackBuilder 为启动的Activity创建一个伪造的返回栈
	// 能够回到桌面
	TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
	// 为Intent添加伪造返回栈
	stackBuilder.addParentStack(ResultActivity.class);
	// Adds the Intent that starts the Activity to the top of the stack
	stackBuilder.addNextIntent(resultIntent);
	PendingIntent resultPendingIntent =
	        stackBuilder.getPendingIntent(
	            0,
	            PendingIntent.FLAG_UPDATE_CURRENT
	        );
	mBuilder.setContentIntent(resultPendingIntent);
	NotificationManager mNotificationManager =
	    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
	// id可以更新Notification
	mNotificationManager.notify(mId, mBuilder.build());
	```
### 2. 应用扩展布局
先创建带有普通视图的NotificationCompat.Builder，调用Builder.setStyle()，参数为以扩展布局为对象，需要Android版本 4.1及以上，4.1之前需要做兼容性处理，参考[Handling compatibility](https://developer.android.com/guide/topics/ui/notifiers/notifications.html#Compatibility)小节。
```java
NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(this)
    .setSmallIcon(R.drawable.notification_icon)
    .setContentTitle("Event tracker")
    .setContentText("Events received")
NotificationCompat.InboxStyle inboxStyle =
        new NotificationCompat.InboxStyle();
String[] events = new String[6];
// 为扩展布局设置标题
inboxStyle.setBigContentTitle("Event tracker details:");
...
//  添加事件到扩展布局中
for (int i=0; i < events.length; i++) {

    inboxStyle.addLine(events[i]);
}
// 添加扩展布局到通知中
mBuilder.setStyle(inBoxStyle);
...
// 发送通知
...
```
### 3. 兼容性处理
为了保证兼容性，使用NotificationCompat或子类来创建通知。在实现通知时，要保证以下几点
- 保证任何版本Android系统的提供功能一致
如：若用addAction() 提供媒体播放的停止和启动播放的控件，要先在应用的Activity中实现此控件。
- 保证通知点击启动Activity来获取Activity中的功能，需要为跳转的Activity创建PendingIntent。
- 如果需要扩展通知功能，保证所有的功能在点击通知时启动的Activity中可用。

## 二、管理通知
在多次发出同一类型的通知时，应该避免创建新的通知。可以更改之前的通知的值或添加值来更新通知。
如：Gmail通过增加未读消息计数并将没封电子邮件的摘要添加到通知，来通知用户新的电子邮件。（这个本成为"堆叠"通知。）

### 1. 更新通知
为了保证通知设置能够更新，通过调用NotificationManager.notify()发出带有ID的通知。为了能够让之前发送通知更新，需要更新或创建一个`NotificationCompat.Builder`，构建`Notification`对象，在调用notify方法时保证ID与之前一致。如果之前发送的通知仍可见，则系统根据新的Notificaiton对象来更新之前通知，否则在界面上创建一个新的通知。
下面展示如何更新通知
```java
mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// 设置ID
int notifyID = 1;
mNotifyBuilder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.ic_notify_status)
        .setContentTitle("New Message")
        .setContentText("You've received new messages");
// 未读消息数量
numMessages = 0;
// 开启数据处理和通知用户的循环
...
// 用相同ID进行通知更新操作
mNotifyBuilder.setContentText(currentText)
        .setNumer(++numMessages);
// 发出通知
mNotificationManager.notify(notifyID , mNotifyBuilder.build());
...
```
### 2. 删除通知
在一下几种情况下，通知会被删除
- 用户通过`全部清除`清除通知（如果通知可以清除时）。当调用Builder的setOnGoing(true)时，可以让通知不会被`全部清除`操作清除掉。
- 用户点击了通知并且在创建通知时调用了[setAutoCancel()](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setAutoCancel(boolean);
- 针对特定的通知ID调用了[cancel()](https://developer.android.com/reference/android/app/NotificationManager.html#cancel(int)，cancel()还会删除当前通知。
- 调用了[cancelAll()](https://developer.android.com/reference/android/app/NotificationManager.html#cancelAll()方法，将删除之前发出的所有通知。

## 三、启动Activity时保存导航顺序
从通知启动Activity时，要保证用户的导航顺序。从通知启动的Activity返回时，用户应该回到应用的正常工作流程界面。因此应该要在全新的任务中来启动Activity并根据启动的Activity要求来设置PendingIntent。
-  常规Activity。
常规的Activity作为应用中正常的工作流的一部分。则需要PendingIntent应该启动新的任务创建新的任务栈。
如：Gmail在点击新的电子邮件的通知时，跳转到消息详情界面。点击返回时，则回到之前的界面。
-  特定Activity
仅在从通知启动时，才会显示的Activity。对于这种情况，PendingIntent需要设置启动的任务为新的任务栈，因为启动的Activity不是应用Activity的流程的一部分，所以不需要创建返回栈。

### 1. 设置常规Activity的PendingIntent
1）. 在AndroidManifest 文件中定义Activity的层次结构
- 支持Android 4.0.3及更低版本。为Activity添加<meta-data>元素为Activity指定Parent，
```xml
     <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity"/>
```
- 支持Android4.1及更高版本。为Activity的activity标签添加android:parentActivityName属性。
最终的xml文件应该是这样的：
```xml
<activity
    android:name=".MainActivity"
    android:label="@string/app_name" >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity
    android:name=".ResultActivity"
    android:parentActivityName=".MainActivity">
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value=".MainActivity"/>
</activity>
```
2）. 为Activity的Intent创建返回栈

a. 创建启动Activity的Intent对象
b. 调用[TaskStackBuilder.create()](https://developer.android.com/reference/android/app/TaskStackBuilder.html#create(android.content.Context)创建栈构造器
c. 调用[addParentStack()](https://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#addParentStack(android.app.Activity)将返回栈添加到栈构造中。对于在Manifest中定义的每个Activity，返回栈含有启动Activity的Intent对象。这个方法还会在新任务中启动栈的标志。

>注：尽管addParentStack()的参数是启动Activity的引用，但是方法不会添加启动Activity的Intent对象，而是在接下来的步骤中

d. 为通知添加启动Activity的Intent，通过调用[addNextIntent()](https://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#addNextIntent(android.content.Intent)方法。使用第一步创建的Intent对象来作为参数
e. 必要情况时调用[TaskStackBuilder.editIntentAt()](https://developer.android.com/reference/android/support/v4/app/TaskStackBuilder.html#editIntentAt(int)为Intent设置参数。有时对于需要保证目标Activity显示有意义的数据。
f. 调用getPendingIntent()获得一个PendingIntent对象，使用这个PengdingIntent对象作为方法setContentIntent()的参数。
```java
// a. 创建启动Activity的Intent对象
Intent resultIntent = new Intent(this , ResultActivity.class);
// b. 创建栈构造器
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
// c. 添加到栈构造器中
stackBuilder.addParentStack(ResultActivity.class);
// d. 添加Intent到栈顶
stackBuilder.addNextIntent(resultIntent);
// f. 获取含有整个返回栈的PendingIntent
PendingIntent resultPendingIntent = stackBuilder.getPengdingIntent(0 , PengdingIntent.FLAG_UPDATE_CURRENT);
...
NotificationCompat.Builder builder = new NotificationCompat.Buidler(this);
builder.setContentIntent(resultPendingIntent);
NotificationManager mNotificationManager = 
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// 发送通知
mNotificationManager.notify(id , builder.build());
```
### 2. 设置特定的Activity PendingIntent
特定的Activity不需要返回栈，所以不需要在Manifest文件中定义Activity的属性同时也不需要调用`addParentStack()`来创建返回栈。但是要为Activity设定Activity的任务栈信息。调用getActivity()方法创建PengdingIntent。
1） 在Manifest中，为Activity的activity元素添加以下属性
- `android:name="activityclass"`
	activity的类名全称
- `android:taskAffinity=""` 任务相关性
	与FLAG_ACTIVITY_NEW_TASK标志位联合使用，保证Activity不会进入应用的默认任务栈。
- `android:excludeFromRecents="true"`
	排除最近的新任务栈，用户无法意外返回到栈中。
```xml
<activity
    android:name=".ResultActivity"
    ...
    android:launchMode="singleTask"         // singleTask启动模式
    android:taskAffinity=""                            // 任务相关性
    android:excludeFromRecents="true"      
</activity>
...
```
2）构建和发送通知
- 创建启动Activity的Intent对象
- 设置Activity的标志位：在一个新的、空的任务栈中启动。调用[setFlags()](https://developer.android.com/reference/android/content/Intent.html#setFlags(int))方法来设置标志位：[FLAG_ACTIVITY_NEW_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK) 和 [FLAG_ACTIVITY_CLEAR_TASK](https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_CLEAR_TASK)
- 设置Intent的可选参数，如需要传递的参数等。
- 调用`PendingIntent`的[getActivity()](https://developer.android.com/reference/android/app/PendingIntent.html#getActivity(android.content.Context, int, android.content.Intent, int))方法创建PendingIntent，使用Builder [setContentIntent](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setContentIntent(android.app.PendingIntent))调用创建的PendingIntent对象。
```java
// 初始化Builder对象
NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
// 创建启动的Activity的Intent
Intent notifyIntent = new Intent(this , ResultActivity.class);
// 设置Acitivity的新的空栈
notifyIntent.setFlag(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
// 创建PendingIntent
PendingIntent notifyPendingIntent = 
    PendingIntent.getActivity(this , 0 , notifyIntent , PendingIntent.FLAG_UPDATE_CURRENT);
// 设置通知的ContentIntent
builder.setContentIntent(notifyPendingIntent);
// 获取到NotificationManager来发送通知
NotificationManager mNotificationManager = 
    (NotificationManager) getSystemService(Context.NOTIFICAITON_SERVICE);
// 发送通知
mNotificationManager.notify(id , builder.build());
```
## 四、通知显示进度条
通知栏可以用进度条来展示进行中的操作进度。如果可以知道操作的总耗时和花费的时间，使用`determinate`的状态来作为指示器。如果不能确切知道操作总长，使用`indeterminate`的状态来作为指示。
Progress指示器一般通过`ProgressBar`来实现。Android 4.0以上调用[setProgress](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))来显示当前的进度，之前版本则必须创建`自定义`的含有`ProgressBar`的布局

### 1. 创建固定的进度指示
显示确定的进度条，可以通过[setProgress(max , progress , `false`)](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean))来为通知添加进度。在处理进度时，来通知进度变化，更新通知栏。操作结束时，进度应该到最大值，常用的方式是调用setProgress()，设定值为`100`。
在操作结束时保留进度条或移除进度条根据业务需求。或者更新进度条的文字来表示操作完成。移除进度条通过调用[setProgress(0, 0, `false`)](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean)的方式
```java
...
mNotificationManager = (NotificationManager)getSystemSerivce(Context.NOTIFICATION_SERVICE);
mBuilder =  new NotificationCompat.Builder(this);
mBuilder.setContentTitle("Picture DownLoad")
    .setContentText("Download in progress");
    .setSmallIcon(R.drawable.ic_notification);
// 在后台线程开启图片下载操作
new Thread( new Runnable(){
    @Override
    public void run(){
        int incr;
        for(incr = 0 ; incr <= 100 ; incr += 5){
            // 设定进度，显示当前进度。
            mBuilder.setProgress(100 , incr , false);
            // 第一次显示进度
            mNotiifcationManager.notifiy(0 , mBuilder.build());
            // 设置线程sleep时间，模拟耗时操作
            try{
                Thread.sleep(5 * 1000);
            } catch (InterruptedException e) {
                Log.d(TAG , "sleep failure");
            }
        }
		// 循环结束后，更新通知
        mBuilder.setContentText("Download complete");
            .setProgress(0 , 0, false);                    // 移除进度
        mNotificationManager.notify(ID , mBuilder.build());
        }
}).start();
```
### 2. 显示加载中的指示器
通过[setProgress( 0 , 0 , `true`)](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setProgress(int, int, boolean)方法来为通知栏添加进行中的操作（前两个参数忽略）。指示样式与进度条类似，但是没有进度指示。
在操作开始时发送通知来显示动画，直到修改通知停止动画。调用setProgress(0 , 0 , `false`)来移除状态指示，通知记住要修改通知栏的文本表示操作结束。
```java
// 明确进度时
mBuilder.setProgress(100 , incr , false);
mNotifyManager.notify(0 , mBuilder.build());
// OR 非明确的进度指示
mBuilder.setProgress(0 , 0 , true);
// 发送通知
mNotifyManager.notify(0 , mBuilder.build());
```
## 五、通知的Metadata 属性
通知可以通过matadata属性来进行排序。可以通过[NotificationCompat.Builder](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)方法来关联metadata属性
- [setCategory](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setCategory(java.lang.String)
	在设备设置了优先级时，如何显示处理通知（如：app会有未接来电、即时消息和闹钟三种通知）。
- [setPriority](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#setPriority(int)
	设置了在`PRIORITY_MAX`和`PRIORITY_HIGH`之间值并且设置了声效或震动时的通知时则会在以浮动通知出现
- [addPerson](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#addPerson(java.lang.String)
	允许添加要通知的人的列表。app可以通过这个方法来告诉系统是否把指定的人的消息通知进行聚合处理或按照重要程度来对通知进行等级排序。

## 六、悬浮的通知。
从Android 5.0（API LEVEL 21）开始，通知可以通过一个小的浮动窗口显示（也被成为悬浮通知）在设备处于活动状态时（也就是设备为锁定并屏幕处于点亮状态）。这些通知显示效果与你的通知形式相似，但是会带有行为按钮。用户可以打开、关闭通知而无需离开当前app。
使用到悬浮通知的情景：
- 用户处于全屏模式
- 或通知处于高优先级且使用了铃声或震动。

## 七、锁屏通知
在Android 5.0及以上版本，可以在锁屏页面来显示通知。使用这个特性来提供媒体播放控制和其他常用的操作。用户可以选择是否在锁屏页来显示通知。
### 1. 设置可见
app可以控制安全锁屏页面通知的显示的细节。调用setVisibility()指定下面的值
- [VISIBILITY_PUBLIC](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html#VISIBILITY_PUBLIC) 显示通知的全部内容
- [VISIBILITY_SECRET](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html#VISIBILITY_SECRET) 不在锁屏页显示通知任何内容
- [VISIBILITY_PROVATE](https://developer.android.com/reference/android/support/v4/app/NotificationCompat.html#VISIBILITY_SECRET) 只显示基本信息，如通知图标、标题。隐藏通知全部内容。
当使用`VISIBILITY_PRIVATE`，可以提供一个通知内容可选的版本来隐藏一些细节。比如：SMS app 可能需要展示通知来显示`你有三个新消息`，但是隐藏消息的具体内容和发件人。提供这样的可选择内容的通知，首先需要用NotificationCompat.Builder创建通知，当创建私有通知对象时，调用·setPublicVersion()`方法添加通知的替代部分。

### 2. 控制媒体的播放
在Android 5.0 显示媒体媒体控制不再基于[RemoteControlClient](https://developer.android.com/reference/android/media/RemoteControlClient.html)。作为替代使用[Notification.MediaStyle](https://developer.android.com/reference/android/app/Notification.MediaStyle.html)的模板，调用[addAction](https://developer.android.com/reference/android/app/Notification.Builder.html#addAction(android.app.Notification.Action))方法来转换操作为可点击的图标。

>注：这个模板和addAction方法不包含在support库中，所以只适应于Android 5.0及更高的版本。

要在Android 5.0上在锁屏页显示媒体播放控制，设置visibility属性为如上描述的VISIBILITY_PUBLIC。然后添加actions并且设置Notification.MediaStyle模板。
```java
Notification notification = new Notification.Builder(context).setVisibility(Notification.VISIBILITY_PUBLIC)
    .setSmallIcon(R.drawable.ic_stat_player)
    .addAction(R.drawable.ic_prev , "Previous" , prevPendingIntent)
    .addAction(R.drawable.ic_pause , "Pause" , pausePendingIntent)
    .addAction(R.drawable.ic_next , "Next" , nextPendingIntent)
    // 应用媒体播放模板
    .setStyle(new Notification.MediaStyle())
    .setShowActionsInCompactView(1)   // 暂停按钮
    .setMediaSession(mMediaSession.getSessionToken())
    .setContextTitle("Wonderfule music")
    .setContextText("My Awesome Band")
    .setLargeIcon(albumArtBitmap)
    .build();
```
>注：RemoteControlClient已经废弃。查看[Media Playback Control](https://developer.android.com/about/versions/android-5.0.html#MediaPlaybackControl)更多关于管理Media和控制播放的新的API

## 八、自定义通知布局
可以用[RemoteView](https://developer.android.com/reference/android/widget/RemoteViews.html)对象来创建自定义通知。自定义通知布局类似于正常的通知，但是基于定义在xml布局文件的RemoteView对象。

自定义通知布局的高度依赖于通知View的高度。常规view布局高度限制为`64dp`，可扩展的View布局限制为`256dp`。
定义自定义通知布局通过inflate xml布局文件创建的RemoteView对象。然后不是通过调用类似setContentTitle()和setContent()方法而是在布局文件中设定内容详情，使用RemoteView中的方法设定View子布局的值。

1. 为通知创建一个独立的XML文件。可以任意命名文件，但是必须是.xml格式文件。
2. 在app中，使用RemoteView方法来定义通知的图标和文字。将RemoteView对象添加到NotificationCompat.Builder对象，通过调用setContent()的方法。避免在RemoteView对象设置背景图片，因为文本颜色可能会与背景颜色相互冲突。

`RemoteView`类同样可以很很容易在自定义的通知布局中添加`Chronometer`和ProgressBar。更多关于创建自定义通知布局的文章，查看[RemoteView](https://developer.android.com/reference/android/widget/RemoteViews.html)参考文档。

>注：当你使用自定义通知布局时，需要考虑到自定义布局在不同设备展示的问题，因为通知栏的空间有很多限制，尽量保证自定义布局简单化并在多种配置设备进行测试。

## 九、自定义通知文本使用样式资源。
保证自定义通知布局使用style资源。通知栏的背景颜色可能会因为不同设备或不同版本，显示效果不一致。所以使用样式资源来帮助你统一管理。从Android 2.3开始，系统为标准通知定义布局文本样式。如果在Android 2.3及以上版本使用相同样式，能够保证在在不同的背景下文本都是可见的。

## 参考
[Notifications](https://developer.android.com/guide/topics/ui/notifiers/notifications.html)

[你应该掌握的Notification](http://blog.csdn.net/xy_nyle/article/details/19853591)
