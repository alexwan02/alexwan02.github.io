---
title: Android：Activity的基础知识及启动过程
date: 2016-06-05 23:14:29
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1467529028/android/8ff23095a2a4e04af26ca63642bfdea3_b.png
tags: android-aosp
categories: android-aosp
---
Activity是Android四个基本组件之一，主要用作Application的界面展示和交互。Activity可以说是Android中应用最多的组件，因此有必要掌握关于Activity的知识。如：Activity的生命周期，Activity的启动模式，更深层次的如Activity的创建过程以及Actvity生命周期对应的方法是如何调用的。
# 一、Activity的生命周期
## 1.正常生命周期
一个Activity正常的流程如下图：
<div>
<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467529027/android/activity_lifecycle.png" style="width:50%">
</div>

1.Activity的完整生命周期：从onCreate()开始到onDestory()结束。Activity在onCreate()中完成一系列初始化的工作，如完成界面布局资源的加载，初始化Activity的需要的数据等等。在onDestory()中释放所有包含的资源。如在结束Actviity时，如果Activity中还有未完成的数据加载的工作，就需要在onDestory()中或之前及时结束。

2.Activity的`可见`生命周期：`可见`生命周期在onStart()与onStop()之间。这段周期中，用户可以看到activity出现在屏幕上，并在onResume()之后，用户可以与界面交互了。当前Activity调用onStop()后，则不再可见，如启动一个新的Activity。在这段生命周期中，你可以添加资源到activity中，向用户展示。如：你在onStart中注册了一个广播，来更新你的UI，然后当activity不可见时，在onStop()中注销这个广播，释放广播资源。在activity整个生命周期中，onStart()和onStop()可能会多次调用，因为activity会在可见和不可见之间多次变化。注：onStart()和onStop()中不要做耗时操作，影响Actvitiy的
显示与停止。

3.Activity前台展示时的生命周期：这段生命周期在调用onResume()和onPause()之间。这段周期，当前Actvitiy位于所有其他Activity顶部（因为Actvitiy就是保存在栈形式的结构中）。Actviity会频繁地在前台的进入和退出之间交互，比如：当回到Android的Home页面、设备处于休眠状态或显示Dialog时，都会调用onPause()，停止当前Actvity的前台状态。因为Actvitiy比较频繁在前台展示的状态交互，在这两方法中，不能进行耗时任务，也是因为新的Actvitiy在显示时，需要暂停前一个Actvitiy的执行，才会调用本身onResume()方法。

## 2.异常时的生命周期

这里还需要考虑到，当系统内存不够用时或旋转屏幕时，Activity会经历哪些生命周期？

<div>
<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467533421/android/restore_instance.png"
	style="width:50%">
</div>

当系统在内存不够用的时候，系统会销毁后台的Activity以来提供足够内存资源给前台的Actvitiy使用，所以在这种情况下，后台的Activity已经被销毁，回到前台时，系统不仅仅只调用onResume()等方法，系统必须重新创建Activity对象。然而用户不会意识到系统销毁了之前Actviity并重新创建了一个新的。因此开发时某时候需要恢复到之前Actvitiy状态：包括数据、界面资源状态等。这时候需要你的Activity覆写Actvity的 onSaveInstanceState()方法，来存储Activity销毁时的数据。

{% codeblock lang:java Actvity.java %}
// 这个方法在onStop()之前调用。
protected void onSaveInstanceState(Bundle outState) {
    // 委托Window保存数据
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
    getApplication().dispatchActivitySaveInstanceState(this, outState);
}
{% endcodeblock %}

系统默认保存当前Activity的视图结构。并在Actvitiy重启后恢复这些数据。首先Actvitiy会调用onSaveInstanceState保存数据，然后Actvitiy会委托Window保存数据，Window委托它上面的顶级容器去保存数据。顶级容器是一个ViewGroup，一般情况下是DecorView。最后顶层容器再去一一通知它的子元素来保存数据。

{% codeblock lang:java PhoneWindow.java %}
// PhoneWindow.java 是Window的实现
@Override
public Bundle saveHierarchyState() {
    Bundle outState = new Bundle();
    if (mContentParent == null) {
        return outState;
    }
    // mContentParent就是ViewGroup
    mContentParent.saveHierarchyState(states);
    ...
    return outState;
}
{% endcodeblock %}


{% codeblock lang:java ViewGroup.java %}

// 分发的每个子View的dispatchSaveInstanceState具体实现
@Override
protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
    super.dispatchSaveInstanceState(container);
    final int count = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < count; i++) {
        View c = children[i];
        if ((c.mViewFlags & PARENT_SAVE_DISABLED_MASK) != PARENT_SAVE_DISABLED) {
            c.dispatchSaveInstanceState(container);
        }
    }
}
{% endcodeblock %}

当Actvitiy被重新创建后，系统会调用onRestoreInstanceState，并把Actvitiy销毁时onSaveInstanceState方法所保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法。
{% codeblock lang:java Actvity.java %}
    // 这个方法在onStart和onPostCreate()之前调用
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        if (mWindow != null) {
            Bundle windowState = savedInstanceState.getBundle(WINDOW_HIERARCHY_TAG);
            if (windowState != null) {
                mWindow.restoreHierarchyState(windowState);
            }
        }
    }
{% endcodeblock %}

在恢复Actvitiy存储时的数据时候，接收的位置可以选择onRestoreInstanceState()或者onCreate，两者区别是onRestoreInstanceState()中的Bundle不为空，onCreate()中的Bundle值可能为空，需要加上判断。

> `注`：onSaveInstanceState()并不能保证正常时被调用。所以不要用来做为数据存储持久化的工作。相反的，当用户离开Activity时应该在onPause()中来存储持久化数据（如数据库数据）。可以简单理解为系统只在Actvitiy异常终止的时候才会调用这两方法，其他情况并不会触发。异常终止：如因为内存不足导致低优先级的Actvitiy被销毁、在旋转屏幕的时候Actvitiy被销毁又被重新创建。

## 3.处理系统配置变化

一些系统配置在运行时可能会发生变化（如：屏幕旋转、键盘变化、语言等）。当这些变化发生的时候，系统会调用onDestroy(),然后立即调用onCreate()。有时候我们并不想要销毁Actvitiy和重新创建，这时候我们可以在AndroidManifest中配置指定Activity的`android:configChanges`的属性即可
{% codeblock lang:xml AndroidManifest.xml %}
<activity 
    ...
    android:configChanges=["mcc", "mnc", "locale",
                         "touchscreen", "keyboard", "keyboardHidden",
                         "navigation", "screenLayout", "fontScale",
                         "uiMode", "orientation", "screenSize",
                         "smallestScreenSize"] 
    ... >
    ...
</activity>
{% endcodeblock %}

常用的三个选项：

	1. local：系统语言变化、
	2. orientation：手机屏幕发生旋转、
	3. keyboardHidden：键盘的可访问性发生了变化，调出键盘。

使用方法

{% codeblock lang:xml AndroidManifest.xml %}
<activity
	android:configChanges="orientation|..."
	... >
	...
</activity>
{% endcodeblock %}

> `注`：screenSize、smallestScreenSize比较特殊，它们的行为与编译选项有关，和运行环境无关。

{% codeblock lang:xml AndroidManifest.xml %}
<!--如果指定的minSdkVersion和targetSdkVersion有一个大于13，为了防止旋转屏幕时Actvitiy重启，除了 -->
<!-- orientation，还需要加上screenSize和smallestScreenSize -->
<uses-sdk
    android:minSdkVersion="..."
    android:targetSdkVersion="..."
    />
{% endcodeblock %}

# 二、Activity的启动模式
为了方便管理Activity，Android引入了Task（任务）和Stack（返回栈）
Task（任务）是用户进行交互的Activity集合。Activity置于类似`后进先出`的对象结构的Stack（返回栈）中。
- 当用户点击Application（应用程序）的Icon时，如果应用的Task没有创建，则系统创建一个新的Task，Application打开Main Activity作为Task的根activity。
- 当启动新的Activity时，创建Activity的实例放入栈，置于栈顶并获取焦点。
- 当Activity进入停止状态，系统保持它的当前状态。
- 当用户点击返回按钮时，栈顶Activity出栈，调用onDestory()销毁实例。前Activity恢复存储的状态并置于栈顶。栈中的Activities不会重新排序，只能够被压入栈和出栈。
- 当用户连续点击返回键，栈中Activity接连出栈，直到回到桌面或Task运行的起始位置。所有activity都被从栈中移除时，则Task销毁。
- 当启动新的Task（如打开新应用）或者点击Home键回到桌面时，任务栈退到后台；当位于后台时，Task中的Activity都会进入停止状态。退到后台的Task的返回栈仍然保持完整性。如：Task A的栈中有三个Activity，用户点击Home键，打开新的应用程序，Task A退到后台。新的应用程序启动，系统创建并启动新的Task B。用户再次回到桌面，打开Task A对应的应用程序，Task A回到前台，Task A中的三个Activity保持完整性。
- Android支持后台多任务；但同时运行多个后台任务，系统可能销毁后台的Activity来回收内存，就导致后台Activity的状态丢失。

## 1. Activity任务管理。
默认情况下Android中的Activity通过`standard`启动模式进入在后进先出的Stack（返回栈）中。有时因为业务需求，需要修改Activity的默认启动模式。如：在新的任务中来启动Activity而不是在返回栈的栈顶创建新的实例；又比如：在启动Activity时，只想要启动Activity已存在的实例；或在用户离开任务时清空除了根Activity外所有的Activity。

修改Activity的默认启动模式有两种方法：
1. 在AndroidManifest中指定Activity的任务和启动模式。
2. 在启动Activity的Intent中加入标志位。

在优先级上第二种高于第一种，就是在Intent中传递Flag标志的方式会覆盖在XML指定启动模式的方式。
标准启动模式，启动Activity时，系统会创建Activity新的实例。Activity可能会多次创建：每个实例可能属于不同任务，一个任务可能有多个实例。

### 1）AndroidManifest中指定Activity的任务和启动模式。
{% codeblock lang:xml AndroidManefest.xml %}
<activity
    android:name=".activity.MainActivity"
    android:taskAffinity="string"
    android:launchMode=["multiple" | "singleTop" |
                  "singleTask" | "singleInstance"]
    android:clearTaskOnLaunch=["true" | "false"]
    android:alwaysRetainTaskState=["true" | "false"]
    android:finishOnTaskLaunch=["true" | "false"] 
    ... >
    ...
</activity>

{% endcodeblock %}
- `taskAffinity`
 taskAffinity表示Activity对应的任务。有相同taskAffinity的Activity理论属于同一个任务。任务自身的Affinity决定
根Activity的Affinity值。taskAffinity的使用场合是什么呢？1.根据taskAffinity重新为Activity选择任务（与
allowTaskReparenting属性配合工作）；2.启动Activity时，Intent使用FLAG_ACTIVITY_NEW_TASK标记，根据
taskAffinity查找或创建一个新Activity对应taskAffinity的任务。默认情况下，应用内所有Activity都具有相同的
taskAffinity,都是从Application继承来，而Application默认taskAffinity值为<manifest>中定义的包名。

- `launchMode`
launchMode表示Activity启动模式。配合Intent的Activity Flags使用。

- `allowTaskReparenting`
Activity是否从启动它的任务中移动到目标任务中，"true"表示可以移动；"false"表示必须保留在启动它的任务中。
如果没有设置，则继承<application>中的属性值，默认false。正常情况下Activity位于启动它的任务中，并度过它的整个生命周期。

- `clearTaskOnLaunch`
标记是否从任务中清除除根Activity的所有Activity，"true"表示清除，"false"表示不清除。默认"false"。这个属性只对根Activity起作用。如果为"true"，每次重新启动应用时，都只看到根Activity，任务中的其他的Activity都会被清除栈。

- `alwaysRetainTaskState`
标记任务是否保持原来的状态，"true"总是保持，"false"不能保证，默认"false"。属性只对根Activity起作用。默认情况下，如果应用在后台停留过长时间，应用再次回到前台时，系统会对应用任务的栈进行清空处理。只保留根Activity。如果根Activity的这个属性为"true"时，应用回到前台时，任务仍然保留所有的Activity。如：浏览器应用打开很多tab页面，在后台停留过长时间，回到前台时，仍然保留这些打开的界面。

- `finishOnTaskLaunch`
finishOnTaskLaunch属性与clearTaskOnLaunch属性类似，不同是它是在操作单个的Activity，而不是整个任务栈。它可以销毁任意Activity包括根Activity。当设置为"true"时，如果用户离开然后回到任务栈，则Activity不再显示。

> 注：多数任务和Activity启动模式应该保持默认值。除非必要情况下，需要改变默认行为。

## 2. 启动模式

Activity有四种启动模式：`standard`，`singleTop`，`singleTask`和`singleInstance`
- `standard`
Activity的默认启动模式，不论栈中是否已存在Activity的实例，都会在创建新的Activity实例，放入栈顶。如下图ActivityA和ActivityB均为standard启动模式。

<div>
	<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467702956/android/android_launchmode_standard.png" style="width:60%;"/>
</div>

- `singleTop`
任务栈顶存在要启动的Activity，系统不会创建新的Activity实例，只调用Activity的onNewIntent()方法。Activity可能被多次实例化，每个Activity实例可能属于不同任务栈，一个任务栈可能有多个实例（仅在返回栈栈顶的Activity不是启动的Activity实例情况下）
假设有任务的返回栈包含ABCD的Activity，A为根Activity，D在栈顶；如果启动D并且D启动模式为"singleTop"，则调用栈顶已经存在的D的方法onNewIntent()，栈内容不变，仍为"ABCD"；如果启动B，B的启动模式为"singleTop"，则会创建新的B实例，并压人栈中。如下图ActivityA和ActivityB均为singleTop启动模式。

<div>
	<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467702956/android/android_launchmode_singleTop.png" style="width:60%;"/>
</div>
> 注：当创建新的Activity，用户点击返回键会返回之前Activity。如果已存在的Activity来处理一个新的Intent对象时，在Intent进入onNewIntent()之前，用户点击返回键无法返回Activity之前的状态。

- `singleTask`
系统查找或创建新Activity对应的任务，已有任务栈时直接向栈中添加Activity的实例；否则创建新的Activity实例作为新任务栈的根。如果指定的任务栈中已经存在Activity的实例，系统只调用Acitivity的onNewIntent()方法，而不是创建新的Activity，同时只能够存在一个相同Activity实例。需要配合`android:taskAffinity`属性来使用。若taskAffinity的值与应用程序一致，新的Activity仍然会在应用程序的默认任务栈中。

<div>
<image
src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467552922/android/diagram_backstack_singletask_multiactivity.png" style="width:60%;"/>
</div>

>注：尽管"singleTask"启动了一个新任务，点击返回键时仍然返回到之前的Activity对应的任务栈。

- `singleInstance`
除了具有"singleTask"的全部特性以外，系统不会在有"singleInstance"启动模式的Activity对应栈中启动任何其他的Activity。具有"singleInstance"启动模式的Activity是栈中唯一的成员，通过这个任务栈启动的Activity都会在指定的任务栈中打开。

<div>
<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467705412/android/android_launchmode_singleInstance.png" style="width:70%"/>
</div>

不管在新的任务中启动Activity还是在启动时的任务栈中启动Activity，点击返回键总是回到之前的Activity。如果你指定Activity的启动模式为singleTask，并在后台任务栈中存在对应的实例。启动这个Acitivity时，就会把整个任务栈带到前台。这时候，返回栈包含所有带到前台的任务栈中所有Activity，并置于栈顶。如singleTask小节图示。

{% codeblock lang:java %}
/**
* 假设有三个Activity：MainAcitivity，ActivityA，ActivityB。MainActivity为应用程序的入口Activity，ActivityA是
* `singleTask`启动模式，任务为`org.alexwan.taskandflag`的Activity，ActivityB与ActivityA一样。在启动MainActivity
* 后，由MainActivity启动ActivityA，ActivityA启动ActivityB。
*/

// 查看运行栈 命令
adb shell dumpsys activity activities | sed -En -e '/Running activities/,/Run #0/p'

// 结果
Running activities (most recent first):
    TaskRecord{d42fd8e #7592 A=org.alexwan.view U=0 sz=2}
        Run #2: ActivityRecord{247d0678 u0 org.alexwan.taskandflag/.ActivityB t7592}
        Run #1: ActivityRecord{168061c4 u0 org.alexwan.taskandflag/.ActivityA t7592}
    TaskRecord{20b981bc #7591 A=org.alexwan.taskandflag U=0 sz=1}
        Run #0: ActivityRecord{618def7 u0 org.alexwan.taskandflag/.MainActivity t7591}
{% endcodeblock %}

## 3.Intent Activity标志
在调用startActivity时，为Intent添加一个标志位决定Activity启动方式。用来修改默认行为.
{% codeblock lang:java Intent.java%}

// 用新任务启动Activity。如果任务中的Activity已运行需要启动的Activity，则直接返回前台状态并调用onNewIntent()方
// 法。与"singleTask"效果相同
FLAG_ACTIVITY_NEW_TASK

// 如果启动的Activity在返回栈栈顶，则直接调用Activity的onNewIntent方法，而不是创建一个新的实例。
// 与"singleTop"效果相同。
FLAG_ACTIVITY_SINGLE_TOP

// 如果启动的Activity已经运行在当前任务栈中，则它的所有顶部Activity都会被销毁，而不是创建一个新Activity实例。调用
// onNewIntent()方法恢复Activity状态。
FLAG_ACTIVITY_CLEAR_TOP

// 启动的Activity不会出现在历史Activity的列表中，在某些情况下我们不希望用户通过历史列表回到Activity时会使用这个标
// 志。与在AndroidManifest中指定Activity的android:excludeFromRecents="true"属性效果相同。
FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

{% endcodeblock %}


> `FLAG_ACTIVITY_CLEAR_TOP`常和`FLAG_ACTIVITY_NEW_TASK`配合使用。当同时使用，这些标志是定位其他栈中存在的Activity和放置在可以享用Intent的位置的方式。这种情况下，被启动Activity的实例如果已经存在，那么系统就会调用它的onNewIntent()方法。如果被启动的Activity采用"standard"启动模式，那么它连同它之上的Activity都要出栈，系统会创建新的Activity实例并置于栈顶。 


## 4. 处理affinities

affinity 表示Activity对应的任务栈值。默认情况下所有Activity继承Application对应的包名所在的任务栈。可以在AndroidManifest中为Activity修改默认affinity值。不同Application可以共享相同affinity属性，同样相同Application中的Activity可以关联不同的affinity属性。
如：
```xml
<activity
    android:name=".ActivityB"
    android:launchMode="singleTask"
    android:taskAffinity="org.alexwan.view" />
```
TaskAffinitys主要配合"singleTask"启动模式和"allowTaskReparenting"属性来使用
- 1）配合"singleTask"启动模式的Activity使用的情景；如通知管理总是在外部的任务栈中启动Activity，而不是作为Application任务栈的一部分。所以通知类的Intent总是用FLAG_ACTIVITY_NEW_TASK在intent的属性中传递给startActivity()。

- 2）配合 allowTaskReparenting 属性使用情景。假设Activity的allowTaskReparenting的值为"true"，这种情况下，在Activity对应的任务栈回到前台，并且已经被其他任务栈启动时，则会从其他任务栈转到Activity对应的任务栈中。比如：在应用程序A中打开浏览器的Activity，Activity初始化时属于应用A的对应的任务栈，当浏览器回到前台时，Activity则从应用A任务栈转到浏览器的任务栈直接显示。


# 三、Activity的启动匹配规则

Intent打开Activity时分为隐式、显式打开Actiivty。

`显式启动`：这种情况启动的Activity为已知，显式Intent也是启动Activity最常用的方式。
```java
Intent intent = new Intent(ActivityA.this , ActivityB.class);
startActivity(intent);
```
`隐式启动`：在未知启动的Activity的情况时，通过action，data，category等IntentFilter属性来过滤匹配要启动的Activity，Activity可能为多个或者没有对应Activity。如：启动分享、打开多媒体相关的Activity等。

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_VIEW);
sendIntent.putExtra(Intent.EXTRA_TEXT , message);
sendIntent.setType("text/plain");
if(sendIntent.resolveActivity(getPackageManager()) != null){
    startActivity(sendIntent);
}
```
> 注：如果没有匹配到相应的Activity时，调用startActivity()时，应用程序直接崩溃。因此在使用隐式Intent时，需要调用resolveActivity()来判断是否有相匹配的Activity来接收Intent，如果没有则不会调用startActivity。

## 1. IntentFilter匹配规则。
```xml
<activity 
    android:name=".activity.ActivityA"
    ...>
    <intent-filter>
        <action android:name="android.intent.action.VIEW">
        ...
        <category android:name="android.intent.category.DEFAULT"/>
        ...
        <data android:scheme="package" />
        ...
    </intent-filter>
</activity>
```
只在有action、category、data都匹配时，Intent才算是匹配成功，如果Activity若声明多个IntentFilter时，只需匹配任意一个则表示匹配成功。

（1） action
一个Intent Filter中可声明0个或多个action，Intent中的action与其中任一action在字符串形式上完全相同（区分大小写），action就算是匹配成功。Intent调用setAction或构造器中传入action为Intent设置action。隐式Intent必须指定action。

（2）category
与action相同，一个Intent Filter可声明多个category或不声明category属性。Intent中的category必须全部匹配Filter中出现的category。Intent若没有指定category，同样能够匹配成功，因为Intent没有指定category时，Android自定为Intent指定默认category值`Intent.CATEGORY_DEFAULT`
```xml
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    ...
</intent-filter>
```

（3）data
与action相同，一个Intent Filter可声明多个data或不声明data属性。
```xml
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http" ... />
    <data android:mimeType="audio/mpeg" android:scheme="http" ... />
    ...
</intent-filter>
```
每个<data>可以指定一个URI和一个数据类型（MIME 媒体类型）。

`URI`
结构：`<scheme>://<host>:<port>/<path>`。如："content://com.example.project:200/folder/subfolder/etc"。

> 注：如果scheme没有指定，则忽略host；host没有指定，则忽略port；如果scheme和host都没有指定，则忽略path。path可以包含星号（*）通配符部分满足path的名称。
URI默认值为content和file。如果filter中没有指定URI，Intent中的URI部分的scheme必须为content或file才能匹配。如果为Intent指定完整的data，必须调用setDataAndType()，单独的调用setData或setType()会重置Data和Type属性。

# 四、Activity的启动过程

<div>
<image src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1467896657/android/%E5%90%AF%E5%8A%A8%E5%BA%94%E7%94%A8%E6%97%B6%E7%9A%84Activity%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.png" style="width:100%">
<div style="text-align:center;"><b>4-1. Android 5.0的启动应用程序时Activity的时序图</b></div>
</div>

应用程序的入口MainActivity的定义

```xml
<activity android:name=".MainActivity"    
      android:label="@string/app_name">    
       <intent-filter>    
        <action android:name="android.intent.action.MAIN" />    
        <category android:name="android.intent.category.LAUNCHER" />    
    </intent-filter>    
</activity>    
```
Android中启动应用程序从桌面启动，系统桌面实际上也是应用程序，对应的是Launcher。

{% codeblock lang:java com.android.launcher2.Launcher.java%}
    boolean startActivitySafely(View v, Intent intent, Object tag) {
        boolean success = false;
        try {
            // 调用Activity的startActivity方法。
            success = startActivity(v, intent, tag);
        } catch (ActivityNotFoundException e) {
            Toast.makeText(this, R.string.activity_not_found, Toast.LENGTH_SHORT).show();
            Log.e(TAG, "Unable to launch. tag=" + tag + " intent=" + intent, e);
        }
        return success;
    }
{% endcodeblock %}

调用Activity的startActivity方法，最终调用Activity的startActivityForResult方法。
{% codeblock lang:java Activity.java %}
    @Override
    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            // Note we want to go through this call for compatibility with
            // applications that may have overridden the method.
            startActivityForResult(intent, -1);
        }
    }

    @Override
    public void startActivityForResult(
            String who, Intent intent, int requestCode, @Nullable Bundle options) {
        ...
        // 1. 创建要启动的Activity实例信息。
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, who,
                intent, requestCode, options);
        if (ar != null) {
            // mMainThread 就是ActivityThread
            mMainThread.sendActivityResult(
                mToken, who, requestCode,
                ar.getResultCode(), ar.getResultData());
        }
        ...
    }
{% endcodeblock %}

`Instrumentation`主要用来监控应用程序的交互操作。只在Activity和ActivityThread有实例，每次在应用程序创建的时候，在ActivityThread中初始化唯一的实例`mInstrumentation`，后续在每个Activity.attach方法中，添加到Activity的中。这里调用execStartActivity执行Activity的启动流程。

{% codeblock lang:java Instrumentation.java%}
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options, UserHandle user) {
        // contextThread 是IBinder对象，主要用来与底层进程之间交互。
        // whoThread 是Launcher的IApplicationThread
        IApplicationThread whoThread = (IApplicationThread) contextThread;
        ...
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess();
            // 获取到ActivityManagerService远程接口即ActivityManagerProxy，调用
            // startActivityAsUser的方法
            int result = ActivityManagerNative.getDefault()
                .startActivityAsUser(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options, user.getIdentifier());
            // 根据返回结果，检测是否成功启动Activity，如果没有抛出异常
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
{% endcodeblock %}

Instrumentation的execStartActivity方法中获得Launcher的IApplicationThread，主要用它与ActivityThread进行进程间通信。获取到ActivityManagerService的远程接口ActivityManagerProxy，调用startActivityAsUser方法。
ActivityManagerProxy定义在ActivityManagerNative中
{% codeblock lang:java ActivityManagerNative.java -> ActivityManagerProxy.java%}
public abstract class ActivityManagerNative extends Binder implements IActivityManager
{
    static public IActivityManager asInterface(IBinder obj) {
        ...
        // 创建ActivityManagerProxy实例
        return new ActivityManagerProxy(obj);
    }
    ...
    static public IActivityManager getDefault() {
        // 单例模式获取到IActivityManager，如果IActivityManager未创建
        // 则调用create方法创建IActivityManager
        return gDefault.get();
    }
    ...
    private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
        protected IActivityManager create() {
            // 查找ActivityManagerService
            IBinder b = ServiceManager.getService("activity");
            // am 就是ActivityManagerProxy
            IActivityManager am = asInterface(b);
            
            return am;
        }
    };
}

class ActivityManagerProxy implements IActivityManager
{
    public ActivityManagerProxy(IBinder remote)
    {
        mRemote = remote;
    }

    public IBinder asBinder()
    {
        return mRemote;
    }
    ...
}

{% endcodeblock %}

调用ActivityManagerProxy的startActivityAsUser方法，通过Binder驱动调用ActivityManagerService的startActivity
```java 

{% codeblock lang:java ActivityManagerProxy.java%}

    public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
            String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options) throws RemoteException {
    ...
    // mRemote 是个Binder，ServiceManager.getService("activity")返回的binder对象
    mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0);
    ...
    return result;
    }
{% endcodeblock %}
调用ActivityManagerNative的onTransact方法。
{% codeblock lang:java ActivityManagerNative.java%}

    @Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
        case START_ACTIVITY_TRANSACTION:
        {
            ...
            IBinder b = data.readStrongBinder();
            // ApplicationThreadProxy
            IApplicationThread app = ApplicationThreadNative.asInterface(b);
            ...
            Intent intent = Intent.CREATOR.createFromParcel(data);
            ...
            // resultTo
            IBinder resultTo = data.readStrongBinder();
            ...
            // ActivityManagerService是ActivityManagerNative的具体实现
            // startActivity也是由ActivityManagerService来执行。
            int result = startActivity(app, callingPackage, intent, resolvedType,
                    resultTo, resultWho, requestCode, startFlags, profilerInfo, options);
            ...
            return true;
        }
{% endcodeblock %}
最终由ActivityManagerService实现调用startActivity
{% codeblock lang:java ActivityManagerService.java%}
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options) {
        // 调用startActivityAsUser
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode, startFlags, profilerInfo, options,
            UserHandle.getCallingUserId());
    }

    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options, int userId) {
        ...
        //  调用ActivityStackSupervisor的startActivityMayWait
        return mStackSupervisor.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, options, false, userId, null, null);
    }

{% endcodeblock %}

`ActivityStackSupervisor`startActivityMayWait

{% codeblock lang:java ActivityStackSupervisor.java%}
    final int startActivityMayWait(IApplicationThread caller, int callingUid,
            String callingPackage, Intent intent, String resolvedType,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode, int startFlags,
            ProfilerInfo profilerInfo, WaitResult outResult, Configuration config,
            Bundle options, boolean ignoreTargetSecurity, int userId,
            IActivityContainer iContainer, TaskRecord inTask) {
        ...
        // Collect information about the target of the Intent.
       // 收集MainActivity信息
        ActivityInfo aInfo = resolveActivity(intent, resolvedType, startFlags, profilerInfo, userId);
        ...
        synchronized (mService) {
            ...
            final ActivityStack stack;
            if (container == null || container.mStack.isOnHomeDisplay()) {
                stack = mFocusedStack;
            } else {
                stack = container.mStack;
            }
            ...
            int res = startActivityLocked(caller, intent, resolvedType, aInfo,
                    voiceSession, voiceInteractor, resultTo, resultWho,
                    requestCode, callingPid, callingUid, callingPackage,
                    realCallingPid, realCallingUid, startFlags, options, ignoreTargetSecurity,
                    componentSpecified, null, container, inTask);
            ...
            return res;
        }
    }

   final int startActivityLocked(IApplicationThread caller,
            Intent intent, String resolvedType, ActivityInfo aInfo,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            IBinder resultTo, String resultWho, int requestCode,
            int callingPid, int callingUid, String callingPackage,
            int realCallingPid, int realCallingUid, int startFlags, Bundle options,
            boolean ignoreTargetSecurity, boolean componentSpecified, ActivityRecord[] outActivity,
            ActivityContainer container, TaskRecord inTask) {
        int err = ActivityManager.START_SUCCESS;
        // 指定进程的所有信息
        ProcessRecord callerApp = null;        
        if (caller != null) {
            callerApp = mService.getRecordForAppLocked(caller);
            if (callerApp != null) {
                callingPid = callerApp.pid;
                callingUid = callerApp.info.uid;
            } else {
                ...
            }
        }
        ...

        final int userId = aInfo != null ? UserHandle.getUserId(aInfo.applicationInfo.uid) : 0;
        // 历史栈中的实体，表示一个Activity
        ActivityRecord sourceRecord = null;
        ActivityRecord resultRecord = null;
        ... 
        // 启动标志位
        final int launchFlags = intent.getFlags();
        ...
        final ActivityStack resultStack = resultRecord == null ? null : resultRecord.task.stack;
        ...
        boolean abort = false;
        ...
        abort |= !mService.mIntentFirewall.checkStartActivity(intent, callingUid,
                callingPid, resolvedType, aInfo.applicationInfo);
        ...
        if (abort) {
            if (resultRecord != null) {
                resultStack.sendActivityResultLocked(-1, resultRecord, resultWho, requestCode,
                        Activity.RESULT_CANCELED, null);
            }
            // We pretend to the caller that it was really started, but
            // they will just get a cancel result.
            ActivityOptions.abort(options);
            return ActivityManager.START_SUCCESS;
        }
        ActivityRecord r = new ActivityRecord(mService, callerApp, callingUid, callingPackage,
                intent, resolvedType, aInfo, mService.mConfiguration, resultRecord, resultWho,
                requestCode, componentSpecified, voiceSession != null, this, container, options);
        ...
        err = startActivityUncheckedLocked(r, sourceRecord, voiceSession, voiceInteractor,
                startFlags, true, options, inTask);

        return err;
    }

   // startActivityUncheckedLocked

    final int startActivityUncheckedLocked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor, int startFlags,
            boolean doResume, Bundle options, TaskRecord inTask) {
        final Intent intent = r.intent;
        final int callingUid = r.launchedFromUid;
        ...
        final boolean launchSingleTop = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TOP;
        final boolean launchSingleInstance = r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE;
        final boolean launchSingleTask = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK;
        // 
        int launchFlags = intent.getFlags();
        ...
        // 如果启动的Activity没有指明是自启动，则在onPause之前调用onUserLeaving
        // launchFlags的FLAG_ACTIVITY_NO_USER_ACTION初始值为0，所以mUserLeaving为true
        mUserLeaving = (launchFlags & Intent.FLAG_ACTIVITY_NO_USER_ACTION) == 0;
        ...
        // 与FLAG_ACTIVITY_NO_USER_ACTION一样intent的flag值为0，所以notTop为null；
        ActivityRecord notTop =
                (launchFlags & Intent.FLAG_ACTIVITY_PREVIOUS_IS_TOP) != 0 ? r : null;

        // If the onlyIfNeeded flag is set, then we can do this if the activity
        // being launched is the same as the one making the call...  or, as
        // a special case, if we do not know the caller then we count the
        // current top activity as the caller.
        if ((startFlags&ActivityManager.START_FLAG_ONLY_IF_NEEDED) != 0) {
            ...
        }

        boolean addingToTask = false;
        TaskRecord reuseTask = null;

        // If the caller is not coming from another activity, but has given us an
        // explicit task into which they would like us to launch the new activity,
        // then let's see about doing that.
        if (sourceRecord == null && inTask != null && inTask.stack != null) {
            ...
        } else {
            inTask = null;
        }

        if (inTask == null) {
            if (sourceRecord == null) {
                ...
            } else if (sourceRecord.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
                ...
            } else if (launchSingleInstance || launchSingleTask) {
                // The activity being started is a single instance...  it always
                // gets launched into its own task.
                // launchFlags 设为Intent.FLAG_ACTIVITY_NEW_TASK
                launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
            }
        }

        ActivityInfo newTaskInfo = null;
        Intent newTaskIntent = null;
        ActivityStack sourceStack;
        if (sourceRecord != null) {
            if (sourceRecord.finishing) {
                ...
            } else {
                sourceStack = sourceRecord.task.stack;
            }
        } else {
            sourceStack = null;
        }

        boolean movedHome = false;
        ActivityStack targetStack;

        intent.setFlags(launchFlags);
        final boolean noAnimation = (launchFlags & Intent.FLAG_ACTIVITY_NO_ANIMATION) != 0;

        // 检索是否存在Task来放Activity；
        if (((launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0 &&
                (launchFlags & Intent.FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
                || launchSingleInstance || launchSingleTask) {
            // Activity的启动模式为launchSingleTask
            if (inTask == null && r.resultTo == null) {
                // 此时inTask为空 ， r.resultTo为空
                // 调用findTaskLocked，因为应用第一次启动，所以检索返回结果为null
                ActivityRecord intentActivity = !launchSingleInstance ?
                        findTaskLocked(r) : findActivityLocked(intent, r.info);
                if (intentActivity != null) {
                    ...
                }
            }
        }
        if (r.packageName != null) {
            // 如果Activity与当前栈顶的Activity一致，判断是否再次启动。
            ActivityStack topStack = mFocusedStack;
            ActivityRecord top = topStack.topRunningNonDelayedActivityLocked(notTop);
            if (top != null && r.resultTo == null) {
                if (top.realActivity.equals(r.realActivity) && top.userId == r.userId) {
                    ...
                }
            }

        } else {
            ...
        }
        boolean newTask = false;
        boolean keepCurTransition = false;

        TaskRecord taskToAffiliate = launchTaskBehind && sourceRecord != null ?
                sourceRecord.task : null;

        // 参数r.resultTo为null，表示Launcher不需要等待启动MainActivity的执行结果
        if (r.resultTo == null && inTask == null && !addingToTask
                && (launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
            newTask = true;
            targetStack = computeStackFocus(r, newTask);
            targetStack.moveToFront("startingNewTask");
            // 创建Task来启动Activity
            if (reuseTask == null) {
                r.setTask(targetStack.createTaskRecord(getNextTaskId(),
                        newTaskInfo != null ? newTaskInfo : r.info,
                        newTaskIntent != null ? newTaskIntent : intent,
                        voiceSession, voiceInteractor, !launchTaskBehind /* toTop */),
                        taskToAffiliate);
                ...
            } else {
                ...
            }
            ...
            if (!movedHome) {
                if ((launchFlags &
                        (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME))
                        == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_TASK_ON_HOME)) {
                    // Caller wants to appear on home activity, so before starting
                    // their own activity we will bring home to the front.
                    r.task.setTaskToReturnTo(HOME_ACTIVITY_TYPE);
                }
            }
        } else if (sourceRecord != null) {
            ...
        } else if (inTask != null) {
            ...
        } else {
            ...
        }

        mService.grantUriPermissionFromIntentLocked(callingUid, r.packageName,
                intent, r.getUriPermissionsLocked(), r.userId);
        ...
        targetStack.mLastPausedActivity = null;
        targetStack.startActivityLocked(r, newTask, doResume, keepCurTransition, options);
        if (!launchTaskBehind) {
            // Don't set focus on an activity that's going to the back.
            mService.setFocusedActivityLocked(r, "startedActivity");
        }
        return ActivityManager.START_SUCCESS;
    }
{% endcodeblock %}

startActivityUncheckedLocked，从命名上可以猜出来方法执行Activity的检查操作。方法中获取到intent的启动标志，对启动模式重新设置，根据标志检索是否是否需要重新创建的Activity的对象、是否需要创建的任务栈、启动时Activity是否需要等待返回值等。然后调用`ActivityStack`的startActivityLocked方法。

{% codeblock lang:java ActivityStack.java %}
    final void startActivityLocked(ActivityRecord r, boolean newTask,
            boolean doResume, boolean keepCurTransition, Bundle options) {
        ...
        TaskRecord task = null;
        if (!newTask) {
            ...
        }

        // Place a new activity at top of stack, so it is next to interact
        // with the user.
        
        ...

        task = r.task;

        // Slot the activity into the history stack and proceed
        // 压入历史栈后处理
        task.addActivityToTop(r);
        task.setFrontOfTask();
        r.putInHistory();
        if (!isHomeStack() || numActivities() > 0) {
            // We want to show the starting preview window if we are
            // switching to a new task, or the next activity's process is
            // not currently running.
            ...
        } else {
            // If this is the first activity, don't do any fancy animations,
            // because there is nothing for it to animate on top of.
            ...
        }
        ...

        if (doResume) {
            mStackSupervisor.resumeTopActivitiesLocked(this, r, options);
        }
    }
{% endcodeblock %}
startActivityLocked主要判断是否需要去进行任务切换时的界面操作。
接着调用ActivityStackSupervisor的resumeTopActivitiesLocked方法。
{% codeblock lang:java ActivityStackSupervisor.java %}
    boolean resumeTopActivitiesLocked(ActivityStack targetStack, ActivityRecord target,
            Bundle targetOptions) {
        if (targetStack == null) {
            targetStack = mFocusedStack;
        }
        // Do targetStack first.
        boolean result = false;
        if (isFrontStack(targetStack)) {
            result = targetStack.resumeTopActivityLocked(target, targetOptions);
        }
        ...
        return result;
    }
{% endcodeblock %}

接着调用ActivityStack的resumeTopActivityLocked方法

{% codeblock lang:java ActivityStack.java %}
    final boolean resumeTopActivityLocked(ActivityRecord prev) {
        return resumeTopActivityLocked(prev, null);
    }

    final boolean resumeTopActivityLocked(ActivityRecord prev, Bundle options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
            // Protect against recursion. 保护防止递归
            mStackSupervisor.inResumeTopActivity = true;
            if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
                mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
                mService.updateSleepIfNeededLocked();
            }
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }
        return result;
    }
    
    // 确保栈顶Activity已经处于Resumed状态
    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, Bundle options) {

        if (!mService.mBooting && !mService.mBooted) {
            //  服务还未启动
            return false;
        }
        ActivityRecord parent = mActivityContainer.mParentActivity;
        if ((parent != null && parent.state != ActivityState.RESUMED) ||
                !mActivityContainer.isAttachedLocked()) {
            // Do not resume this stack if its parent is not resumed.
            return false;
        }
        cancelInitializingActivities();

        // 找到栈顶ActivityRecord
        final ActivityRecord next = topRunningActivityLocked(null);

        // Remember how we'll process this pause/resume situation, and ensure
        // that the state is reset however we wind up proceeding.
        // mUserLeaving保存在本地，重新设置为false
        final boolean userLeaving = mStackSupervisor.mUserLeaving;
        mStackSupervisor.mUserLeaving = false;

        final TaskRecord prevTask = prev != null ? prev.task : null;
        if (next == null) {
            // There are no more activities!
            ...
        }

        next.delayedResume = false;

        // 如果栈顶Activity已经处于resume状态，直接返回
        if (mResumedActivity == next && next.state == ActivityState.RESUMED &&
                    mStackSupervisor.allResumedActivitiesComplete()) {
            ...
            return false;
        }
        ...

        // 如果休眠状态并且没有需要resume的activity，栈顶activity处于暂停状态，直接返回
        if (mService.isSleepingOrShuttingDown()
                && mLastPausedActivity == next
                && mStackSupervisor.allPausedActivitiesComplete()) {
            ...
            return false;
        }

        // 验证activity的拥有者已经启动
        if (mService.mStartedUsers.get(next.userId) == null) {
            ...
            return false;
        }
        // 如果activity正在等待停止或休眠，则从停止或休眠队列中移除这个activity
        // 因为不适应activity了
        mStackSupervisor.mStoppingActivities.remove(next);
        mStackSupervisor.mGoingToSleepActivities.remove(next);
        next.sleeping = false;
        mStackSupervisor.mWaitingVisibleActivities.remove(next);
        ...
        // 如果现在正在暂停一个activity，返回等待则进入等待状态。
        if (!mStackSupervisor.allPausedActivitiesComplete()) {
            ...
            return false;
        }
        ... 
        mStackSupervisor.setLaunchSource(next.info.applicationInfo.uid);

        // 在activity调用resumed之前需要暂停之前的activity
        boolean dontWaitForPause = (next.info.flags&ActivityInfo.FLAG_RESUME_WHILE_PAUSING) != 0;
        boolean pausing = mStackSupervisor.pauseBackStacks(userLeaving, true, dontWaitForPause);
        if (mResumedActivity != null) {
            ...
            // 开始暂停之前的Activity，mResumedActivity指定启动时的Activity
            pausing |= startPausingLocked(userLeaving, false, true, dontWaitForPause);
        }
        if (pausing) {
            ...
            // At this point we want to put the upcoming activity's process
            // at the top of the LRU list, since we know we will be needing it
            // very soon and it would be a waste to let it get killed if it
            // happens to be sitting towards the end.
            // 如果正在暂停之前的activity，现在将要启动的activity的进程放在LRU列表的顶部，因为要很快要需要这个参数
            // 
            if (next.app != null && next.app.thread != null) {
                mService.updateLruProcessLocked(next.app, true, null);
            }
            ...
            return true;
        }
        ...
        return true;
    }

    // 执行Activity暂停操作
    /**
    * 保存mResumedActivity到本地变量prev中，在本文中mResumedActivity对应的就是Launcher。
    * 调用 Launcher对应的ApplicationThread对象的远程接口，也就是ApplicationThreadProxy。执行
    * ApplicationThreadProxy的schedulePauseActivity方法，经过底层驱动Binder，通知Launcher进入
    * Paused状态。
    */
    final boolean startPausingLocked(boolean userLeaving, boolean uiSleeping, boolean resuming,
            boolean dontWait) {
        if (mPausingActivity != null) {
            // 如果有暂停中的Activity
            ...
        }
        ActivityRecord prev = mResumedActivity;
        if (prev == null) {
            ...
            return false;
        }

        if (mActivityContainer.mParentActivity == null) {
            // Top level stack, not a child. Look for child stacks.
            mStackSupervisor.pauseChildStacks(prev, userLeaving, uiSleeping, resuming, dontWait);
        }
        ...
        // 将mResumedActivity置空，mResumedActivity赋值给mPausingActivity
        mResumedActivity = null;
        mPausingActivity = prev;
        mLastPausedActivity = prev;
        mLastNoHistoryActivity = (prev.intent.getFlags() & Intent.FLAG_ACTIVITY_NO_HISTORY) != 0
                || (prev.info.flags & ActivityInfo.FLAG_NO_HISTORY) != 0 ? prev : null;
        prev.state = ActivityState.PAUSING;
        prev.task.touchActiveTime();
        clearLaunchTime(prev);
        // 启动的Activity
        final ActivityRecord next = mStackSupervisor.topRunningActivityLocked();
        ...
        if (prev.app != null && prev.app.thread != null) {
            ...
            try {
                ...
                prev.app.thread.schedulePauseActivity(prev.appToken, prev.finishing,
                        userLeaving, prev.configChangeFlags, dontWait);
            } catch (Exception e) {
                ...
                mPausingActivity = null;
                mLastPausedActivity = null;
                mLastNoHistoryActivity = null;
            }
        } else {
            mPausingActivity = null;
            mLastPausedActivity = null;
            mLastNoHistoryActivity = null;
        }
        ...
    }
{% endcodeblock %}

调用Launcher的ApplicationThread远程接口ApplicationThreadProxy的schedulePauseActivity方法

{% codeblock lang:java ApplicationThreadProxy.java%}
    public final void schedulePauseActivity(IBinder token, boolean finished,
            boolean userLeaving, int configChanges, boolean dontReport) throws RemoteException {
        Parcel data = Parcel.obtain();
        data.writeInterfaceToken(IApplicationThread.descriptor);
        data.writeStrongBinder(token);
        data.writeInt(finished ? 1 : 0);
        data.writeInt(userLeaving ? 1 :0);
        data.writeInt(configChanges);
        data.writeInt(dontReport ? 1 : 0);
        // 
        mRemote.transact(SCHEDULE_PAUSE_ACTIVITY_TRANSACTION, data, null,
                IBinder.FLAG_ONEWAY);
        data.recycle();
    }
{% endcodeblock %}

经过Binder驱动通知ApplicationThread指定对应的schedulePauseActivity方法。ApplicationThread为ActivityThread的内部类。

{% codeblock lang:java ActivityThreadjava -> ApplicationThread.java ， H.java%}
    private class ApplicationThread extends ApplicationThreadNative {
        public final void schedulePauseActivity(IBinder token, boolean finished,
                boolean userLeaving, int configChanges, boolean dontReport) {
            sendMessage(
                    finished ? H.PAUSE_ACTIVITY_FINISHING : H.PAUSE_ACTIVITY,
                    token,
                    (userLeaving ? 1 : 0) | (dontReport ? 2 : 0),
                    configChanges);
        }
    }

    // 调用ActivityThread sendMessage发送Message，在H中处理Message
    private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
        ...
        Message msg = Message.obtain();
        msg.what = what;
        msg.obj = obj;
        msg.arg1 = arg1;   // 1
        msg.arg2 = arg2;  // configChanges
        if (async) {
            msg.setAsynchronous(true);
        }
        mH.sendMessage(msg);
    }

    private class H extends Handler {
        public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
        ...
        public void handleMessage(Message msg) {
                ...
                case PAUSE_ACTIVITY:
                    ...
                    // arg1 = 1 , 
                    handlePauseActivity((IBinder)msg.obj, false, (msg.arg1&1) != 0, msg.arg2,
                            (msg.arg1&2) != 0);
                    ...
                    break;
                ...
        }
    }
    //  最后调用handlePauseActivity
    private void handlePauseActivity(IBinder token, boolean finished,
            boolean userLeaving, int configChanges, boolean dontReport) {
        ActivityClientRecord r = mActivities.get(token);
        if (r != null) {
            ...
            if (userLeaving) {
                // 1. 通知Activity，用户将要离开界面
                performUserLeavingActivity(r);
            }

            r.activity.mConfigChangeFlags |= configChanges;
            // 2. 调用Activity的onPaused方法。
            performPauseActivity(token, finished, r.isPreHoneycomb());

            // Make sure any pending writes are now committed.
            if (r.isPreHoneycomb()) {
                QueuedWork.waitToFinish();
            }

            // Tell the activity manager we have paused.
            // dontReport = (msg.arg1&2) != 0 ; dontReport的值为false
            if (!dontReport) {
                try {
                    // 3. 调用ActivityManager的远程服务接口 ActivityManagerProxy
                    // 通知ActivityManagerService，当前activity已进入暂停状态，可以执行未完成任务。
                    ActivityManagerNative.getDefault().activityPaused(token);
                } catch (RemoteException ex) {
                }
            }
            mSomeActivitiesChanged = true;
        }
    }
{% endcodeblock %}

ActivityThread在方法`schedulePauseActivity`只简单的调用了sendMessage()方法。然后调用ActivityThread内部类H的handlePauseActivity方法，在handlePauseActivity中做了以下的任务：1. 将Binder引用的token转成ActivityRecord的远程接口ActivityClientRecord。如果userLeaving为true时，则调用performUserLeavingActivity来通知Activity，用户将要离开界面。3. 通知ActivityManagerService，当前activity已进入暂停状态，可以执行未完成任务。这里表示启动MainActivity。

{% codeblock lang:java ActivityManagerNative.java -> ActivityManagerProxy.java%}
class ActivityManagerProxy implements IActivityManager{
    ...
    public void activityPaused(IBinder token) throws RemoteException
    {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IActivityManager.descriptor);
        data.writeStrongBinder(token);
        mRemote.transact(ACTIVITY_PAUSED_TRANSACTION, data, reply, 0);
        reply.readException();
        data.recycle();
        reply.recycle();
    }
    ...
}
{% endcodeblock %}

经过Binder驱动调用ActivityManagerService.activityPaused方法
{% codeblock lang:java ActivityManager.java %}
    @Override
    public final void activityPaused(IBinder token) {
        final long origId = Binder.clearCallingIdentity();
        synchronized(this) {
            ActivityStack stack = ActivityRecord.getStackLocked(token);
            if (stack != null) {
                stack.activityPausedLocked(token, false);
            }
        }
        ...
    }
{% endcodeblock %}

调用ActivityStack的activityPausedLocked 的方法

{% codeblock lang:java ActivityStack.java %}
    final void activityPausedLocked(IBinder token, boolean timeout) {
        ...
        final ActivityRecord r = isInStackLocked(token);
        if (r != null) {
            ...
            // startPausingLocked时将当前Activity保存在mPausingActivity中。
            if (mPausingActivity == r) {
                ...
                completePauseLocked(true);
            } else {
                ... 
            }
        }
    }

    final void activityPausedLocked(IBinder token, boolean timeout) {
        ...

        final ActivityRecord r = isInStackLocked(token);
        if (r != null) {
            ...
            // startPausingLocked时将当前Activity保存在mPausingActivity中。
            if (mPausingActivity == r) {
                ...
                completePauseLocked(true);
            } else {
                ...
                
            }
        }
    }
    
    // 完成Activity的暂停任务
    private void completePauseLocked(boolean resumeNext) {
        ActivityRecord prev = mPausingActivity;
        if (prev != null) {
            prev.state = ActivityState.PAUSED;
            ...
            // 在activity暂停之前，会暂时冻住屏幕。这时Activity不再可见，则解除冻结状态
            prev.stopFreezingScreenLocked(true /*force*/);
            mPausingActivity = null;
        }
        // resumeNext值为true
        if (resumeNext) {
            // resume 新的activity
            final ActivityStack topStack = mStackSupervisor.getFocusedStack();
            if (!mService.isSleepingOrShuttingDown()) {
                // 如果没有休眠或关机
                mStackSupervisor.resumeTopActivitiesLocked(topStack, prev, null);
            } else {
                ...
            }
        }

        if (prev != null) {
            // 恢复按键分发
            prev.resumeKeyDispatchingLocked();
            ...
        }

        // 栈内容变化时发送通知。
        mService.notifyTaskStackChangedLocked();
    }    
{% endcodeblock %}

在方法completePauseLocked中：如果mPausingActivity不为空，则mPausingActivity需要置空。而mPausingActivity是在之前调用`startPausingLocked`保存的Launcher的实例，现在不需要这个临时对象了。获取的启动应用的栈信息，调用mStackSupervisor的resumeTopActivitiesLocked方法
{%  codeblock lang:java ActivityStackSupervisor.java %}
    boolean resumeTopActivitiesLocked(ActivityStack targetStack, ActivityRecord target,
            Bundle targetOptions) {
        ...     
        boolean result = false;
        if (isFrontStack(targetStack)) {
            result = targetStack.resumeTopActivityLocked(target, targetOptions);
        }
        ...
        return result;
    }
{% endcodeblock %}

又执行了ActivityStack的resumeTopActivityLocked方法中。主要用作保护防止无限递归

{% codeblock lang:java ActivityStack.java %}
    final boolean resumeTopActivityLocked(ActivityRecord prev, Bundle options) {
        ...
        boolean result = false;
        try {
            // Protect against recursion. 
            ...
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            ...
        }
        return result;
    }

    private boolean resumeTopActivityInnerLocked(ActivityRecord prev, Bundle options) {
        ...
        cancelInitializingActivities();

        // Find the first activity that is not finishing. 
        // 1. 找到栈顶Activity，也就是要启动的Activity。
        final ActivityRecord next = topRunningActivityLocked(null);

        ...
        final boolean userLeaving = mStackSupervisor.mUserLeaving;
        mStackSupervisor.mUserLeaving = false;
        ... 
        final TaskRecord prevTask = prev != null ? prev.task : null;
        if (next == null) {
            ...
            return isOnHomeDisplay() &&
                    mStackSupervisor.resumeHomeStackTask(returnTaskType, prev, reason);
        }

        next.delayedResume = false;
        // 2. mResumedActivity此时为空
        if (mResumedActivity == next && next.state == ActivityState.RESUMED &&
                    mStackSupervisor.allResumedActivitiesComplete()) {
            ...
            return false;
        }

        final TaskRecord nextTask = next.task;
        if (prevTask != null && prevTask.stack == this &&
                prevTask.isOverHomeStack() && prev.finishing && prev.frontOfTask) {
            ...
        }

        // 休眠状态或没有找到Activity需要执行resume直接返回
        if (mService.isSleepingOrShuttingDown()
                && mLastPausedActivity == next
                && mStackSupervisor.allPausedActivitiesComplete()) {
            ...
            return false;
        }

        ...

        // 暂停Activity已经执行，跳过
        if (!mStackSupervisor.allPausedActivitiesComplete()) {
            ...
            return false;
        }
        // 设置启动信息
        mStackSupervisor.setLaunchSource(next.info.applicationInfo.uid);

        // launcher已经暂停，跳过
        boolean dontWaitForPause = (next.info.flags&ActivityInfo.FLAG_RESUME_WHILE_PAUSING) != 0;
        boolean pausing = mStackSupervisor.pauseBackStacks(userLeaving, true, dontWaitForPause);
        if (mResumedActivity != null) {
            ...
        }
        if (pausing) {
            ...
            return true;
        }
        
        // 并不是休眠状态
        if (mService.isSleeping() && mLastNoHistoryActivity != null &&
                !mLastNoHistoryActivity.finishing) {
            ...
        }

        if (prev != null && prev != next) {
            if (!mStackSupervisor.mWaitingVisibleActivities.contains(prev)
                    && next != null && !next.nowVisible) {
                // 将要启动的Acitivity添加等待显示的列表中
                mStackSupervisor.mWaitingVisibleActivities.add(prev);
                
            } else {

                // 如果之前的Acitivity消失了，执行这段代码。
                ...
            }
        }

        // Launching this app's activity, make sure the app is no longer
        // considered stopped.
        // 启动app，
        try {
            AppGlobals.getPackageManager().setPackageStoppedState(
                    next.packageName, false, next.userId); /* TODO: Verify if correct userid */
        } catch (RemoteException e1) {
        } catch (IllegalArgumentException e) {
           ...
        }

        // 开始启动新的Activity，通知WindowManager之前的Activity将会很快消失。这样
        // 在计算需求的屏幕方向时忽略它
        boolean anim = true;
        if (prev != null) {
            // 为Activity准备Window基本参数配置，是否显示启动动画等
            if (prev.finishing) {
                ... 
                if (mNoAnimActivities.contains(next)) {
                    anim = false;
                    ...
                }
                ...
            } else {
                ...
            }
            ...
        } else {
            ...
        }
        // 执行resume时的动画参数
        Bundle resumeAnimOptions = null;
        if (anim) {
            // 
            ActivityOptions opts = next.getOptionsForTargetActivityLocked();
            if (opts != null) {
                resumeAnimOptions = opts.toBundle();
            }
            ...
        } else {
            ...
        }
        ...
        // 3. 获取最近的Activity栈信息
        ActivityStack lastStack = mStackSupervisor.getLastStack();
        if (next.app != null && next.app.thread != null) {
            // 因为是从Launcher中第一次启动程序，所以程序没有这些进程和主线程信息。
            ...
        } else {
            
            // 需要重启Activity：如正常启动程序或闪退后启动程序
            if (!next.hasBeenLaunched) {
                // 之前没有启动过
                next.hasBeenLaunched = true;
            } else {
                ...
            }
            // 开始执行应用程序的进程和ActivityThread等重要参数的初始化操作
            mStackSupervisor.startSpecificActivityLocked(next, true, true);
        }
        ...
        return true;
    }

{% endcodeblock %}

resumeTopActivityInnerLocked中执行的任务
1）取出栈顶Activity，也就是要启动的Activity。
2）Launcher此时已经处于Pasued状态，所以此时mResumedActivity为null，mLastPausedActivity为Launcher。
3）因为应用还未启动所以MainActivity的ActivityRecord的app和thread属性还未初始化，都为空，则调用ActivityStackSupervisor的startSpecificActivityLocked方法初始化应用的重要变量：ActivityThread等。

{% codeblock lang:java ActivityStackSupervisor.java %}
    void startSpecificActivityLocked(ActivityRecord r,
            boolean andResume, boolean checkConfig) {
        // Is this activity's application already running?
        // activity的application是否已经运行
        ProcessRecord app = mService.getProcessRecordLocked(r.processName,
                r.info.applicationInfo.uid, true);
     
        r.task.stack.setLaunchTime(r);

        if (app != null && app.thread != null) {
            // 如果程序已经运行了，则执行realStartActivityLocked的流程
            try {
                ...
                realStartActivityLocked(r, app, andResume, checkConfig);
                return;
            } catch (RemoteException e) {
                ...
            }
            ...
        }
        // 因为这时的应用还未启动则执行startProcessLocked方法，开启新的进程
        mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
                "activity", r.intent.getComponent(), false, false, true);
    }
{% endcodeblock %}

因为首次的应用，所以取到的ProcessRecord为null。默认情况下，ActivityRecord中的进程名processName对应的就是在`AndroidManifest`中声明包名。调用ActivityManagerService的startProcessLocked方法来执行初始化任务。
{% codeblock lang:java ActivityManagerService.java %}
    final ProcessRecord startProcessLocked(String processName, ApplicationInfo info,
            boolean knownToBeDead, int intentFlags, String hostingType, ComponentName hostingName,
            boolean allowWhileBooting, boolean isolated, int isolatedUid, boolean keepIfLarge,
            String abiOverride, String entryPoint, String[] entryPointArgs, Runnable crashHandler) {
        ...
        ProcessRecord app;
        // isolated为true
        if (!isolated) {
            ...
        } else {
            ...
            // 如果是单独进程，则不能重用已存在的进程
            app = null;
        }
        ...
        // 如果存在一个Application；
        // 调用者不认为已经死亡或没有线程对象时我们认为没有崩溃；
        // 或分配了一个进程id时，则认为正在启动或已经运行；
        // 这三种情况下，不会做任何事
        if (app != null && app.pid > 0) {
            if (!knownToBeDead || app.thread == null) {
                // 已经运行App或等待出现（已经有进程id，但是还没有线程），则保留应用
                ...
                // 如果是进程的新包，则添加新包到列表中
                app.addPackage(info.packageName, info.versionCode, mProcessStats);
                ...
                return app;
            }

            // app添加到之前的进程中，则清空进程
            ...
        }
        
        // host name
        String hostingNameStr = hostingName != null
                ? hostingName.flattenToShortString() : null;
        ...
        if (app == null) {
            ...
            // 为app 创建新的进程
            app = newProcessRecordLocked(info, processName, isolated, isolatedUid);
            if (app == null) {
                return null;
            }
            app.crashHandler = crashHandler;
            ...
        } else {
            // 如果是进程的新包，则添加新包到列表中
            app.addPackage(info.packageName, info.versionCode, mProcessStats);
            ...
        }

        // 推迟进程启动直到系统准备好。
        if (!mProcessesReady
                && !isAllowedWhileBooting(info)
                && !allowWhileBooting) {
            ...
        }
        // 开启进程
        startProcessLocked(
                app, hostingType, hostingNameStr, abiOverride, entryPoint, entryPointArgs);
        ...
        return (app.pid != 0) ? app : null;
    }

     final ProcessRecord newProcessRecordLocked(ApplicationInfo info, String customProcess,
            boolean isolated, int isolatedUid) {
        // 进程命名 ： processName+uid
        String proc = customProcess != null ? customProcess : info.processName;
        BatteryStatsImpl stats = mBatteryStatsService.getActiveStatistics();
        final int userId = UserHandle.getUserId(info.uid);
        int uid = info.uid;
        if (isolated) {
            // isolatedUid 为 0
            if (isolatedUid == 0) {
                int stepsLeft = Process.LAST_ISOLATED_UID - Process.FIRST_ISOLATED_UID + 1;
                while (true) {
                    ...
                    uid = UserHandle.getUid(userId, mNextIsolatedProcessUid);
                    mNextIsolatedProcessUid++;
                    if (mIsolatedProcesses.indexOfKey(uid) < 0) {
                        // No process for this uid, use it.
                        break;
                    }
                    ...
                }
            } else {
                ...
                uid = isolatedUid;
            }
        }
        final ProcessRecord r = new ProcessRecord(stats, info, proc, uid);
        ...
        addProcessNameLocked(r);
        return r;
    }

    private final void addProcessNameLocked(ProcessRecord proc) {
        // 清空旧进程
        ProcessRecord old = removeProcessNameLocked(proc.processName, proc.uid);
        if (old == proc && proc.persistent) {
            // We are re-adding a persistent process.  Whatevs!  Just leave it there.
            ...
        } else if (old != null) {
            ...
        }
        ...
        // mProcessNames添加进程信息
        mProcessNames.put(proc.processName, proc.uid, proc);
        if (proc.isolated) {
            mIsolatedProcesses.put(proc.uid, proc);
        }
    }

{% endcodeblock %}

ActivityManagerService的startProcessLocked方法：检查是否有对应的进程存在，如果没有进程（如启动新应用时），则
newProcessRecordLocked初始化进程基本参数：pid，uid，进程名等等。并保存在mProcessNames全局变量中。然后执行`startProcessLocked`方法，进入下一步。

{% codeblock lang:java ActivityManagerService.java%}
    private final void startProcessLocked(ProcessRecord app, String hostingType,
            String hostingNameStr, String abiOverride, String entryPoint, String[] entryPointArgs) {
        ...
        if (app.pid > 0 && app.pid != MY_PID) {
            ...
            app.setPid(0);
        }
        ...
        mProcessesOnHold.remove(app);
        ...

        try {
            ...
            int uid = app.uid;
            int[] gids = null;
            int mountExternal = Zygote.MOUNT_EXTERNAL_NONE;
            if (!app.isolated) {
                ...
            }
            ...
            if (mFactoryTest != FactoryTest.FACTORY_TEST_OFF) {
                if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL
                        && mTopComponent != null
                        && app.processName.equals(mTopComponent.getPackageName())) {
                    uid = 0;
                }
                if (mFactoryTest == FactoryTest.FACTORY_TEST_HIGH_LEVEL
                        && (app.info.flags&ApplicationInfo.FLAG_FACTORY_TEST) != 0) {
                    uid = 0;
                }
            }
            ... 
            // debug 标志
            int debugFlags = 0;
            ...
            String requiredAbi = (abiOverride != null) ? abiOverride : app.info.primaryCpuAbi;
            if (requiredAbi == null) {
                requiredAbi = Build.SUPPORTED_ABIS[0];
            }
            String instructionSet = null;
            if (app.info.primaryCpuAbi != null) {
                instructionSet = VMRuntime.getInstructionSet(app.info.primaryCpuAbi);
            }
            app.gids = gids;
            app.requiredAbi = requiredAbi;
            app.instructionSet = instructionSet;

            // Start the process.  It will either succeed and return a result containing
            // the PID of the new process, or else throw a RuntimeException.
            // 启动进程，成功则返回含有新进程pid信息的结构，否则抛出异常。
            boolean isActivityProcess = (entryPoint == null);
            // "android.app.ActivityThread"
            if (entryPoint == null) entryPoint = "android.app.ActivityThread";
            ...
            // 创建新的进程，
            Process.ProcessStartResult startResult = Process.start(entryPoint,
                    app.processName, uid, uid, gids, debugFlags, mountExternal,
                    app.info.targetSdkVersion, app.info.seinfo, requiredAbi, instructionSet,
                    app.info.dataDir, entryPointArgs);
            ...
        } catch (RuntimeException e) {
            
            ...
        }
    }
{% endcodeblock %}

Process的start方法向Zygote发送请求，传入"android.app.ActivityThread"字符串参数，通过Zygote执行fork子进程，初始化应用最终调用ActivityThread的main方法。

{% codeblock lang:java ActivityThread.java%}
    public static void main(String[] args) {
        ...
        SamplingProfilerIntegration.start();
        ...
        Environment.initForCurrentUser();
        ...
        AndroidKeyStoreProvider.install();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");
        // 初始化MainLooper
        Looper.prepareMainLooper();
        // 创建ActivityThread
        ActivityThread thread = new ActivityThread();
        // attach 调用attach方法
        thread.attach(false);
        // 主线程的sMainThreadHandler
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        ...
        // 开启主线程循环
        Looper.loop();
        ...
    }

    private void attach(boolean system) {
        sCurrentActivityThread = this;
        // system值为false
        mSystemThread = system;
        if (!system) {
            ...
            RuntimeInit.setApplicationObject(mAppThread.asBinder());
            final IActivityManager mgr = ActivityManagerNative.getDefault();
            try {
                // 调用ActivityManagerProxy进行远程通信。
                mgr.attachApplication(mAppThread);
            } catch (RemoteException ex) {
                // Ignore
            }
            ...
        } else {
            // 不能设置Application对象。如果系统崩溃了，直接结束。
            ...
        }
        ...
    }
{% endcodeblock %}

ActivityThread的main方法是整个应用程序启动的入口。执行了ActivityThread、主线程消息Looper的初始化操作。然后调用ActivityManagerProxy的attachApplication方法通过Binder驱动通知ActivityManagerService的attachApplication执行应用启动的后续操作。
{% codeblock lang:java ActivityManagerService%}
    @Override
    public final void attachApplication(IApplicationThread thread) {
        synchronized (this) {
            ...
            attachApplicationLocked(thread, callingPid);
            ...
        }
    }

    private final boolean attachApplicationLocked(IApplicationThread thread,
            int pid) {

        // Find the application record that is being attached...  either via
        // the pid if we are running in multiple processes, or just pull the
        // next app record if we are emulating process with anonymous threads.

        // 检索需要附加的application信息。
        // 1. 运行在多进程中pid信息;2. 或拉取用匿名线程模拟的进程启动的app信息。
        ProcessRecord app;
        if (pid != MY_PID && pid >= 0) {
            synchronized (mPidsSelfLocked) {
                app = mPidsSelfLocked.get(pid);
            }
        } else {
            app = null;
        }

        if (app == null) {
            ...
            // 未找到application
            return false;
        }

        // If this application record is still attached to a previous
        // process, clean it up now.
        // 如果application record仍附加在之前的进程，则结束application
        if (app.thread != null) {
            handleAppDiedLocked(app, true, true);
        }
        // 进程名：
        final String processName = app.processName;
        try {
            AppDeathRecipient adr = new AppDeathRecipient(
                    app, pid, thread);
            thread.asBinder().linkToDeath(adr, 0);
            app.deathRecipient = adr;
        } catch (RemoteException e) {
            ...
            return false;
        }
        ...
        // 激活进程状态
        app.makeActive(thread, mProcessStats);
        app.curAdj = app.setAdj = -100;
        app.curSchedGroup = app.setSchedGroup = Process.THREAD_GROUP_DEFAULT;
        app.forcingToForeground = null;
        updateProcessForegroundLocked(app, false, false);
        app.hasShownUi = false;
        app.debugging = false;
        ...
        boolean normalMode = mProcessesReady || isAllowedWhileBooting(app.info);
        List<ProviderInfo> providers = normalMode ? generateApplicationProvidersLocked(app) : null;
        ...
        try {
            ...
            ApplicationInfo appInfo = app.instrumentationInfo != null
                    ? app.instrumentationInfo : app.info;
            app.compat = compatibilityInfoForPackageLocked(appInfo);
            if (profileFd != null) {
                profileFd = profileFd.dup();
            }
            ProfilerInfo profilerInfo = profileFile == null ? null
                    : new ProfilerInfo(profileFile, profileFd, samplingInterval, profileAutoStop);
            // 1. 初始化应用中对应系统信息，
            thread.bindApplication(processName, appInfo, providers, app.instrumentationClass,
                    profilerInfo, app.instrumentationArguments, app.instrumentationWatcher,
                    app.instrumentationUiAutomationConnection, testMode, enableOpenGlTrace,
                    enableTrackAllocation, isRestrictedBackupMode || !normalMode, app.persistent,
                    new Configuration(mConfiguration), app.compat,
                    getCommonServicesLocked(app.isolated),
                    mCoreSettingsObserver.getCoreSettingsLocked());
            updateLruProcessLocked(app, false, null);
            ...
        } catch (Exception e) {
            ...
        }
        ...
        boolean badApp = false;
        boolean didSomething = false;
        ... 
        // 查看进程中是否有栈顶Activity等待运行
        if (normalMode) {
            try {
                // 2. 开始执行应用的MainActivity的启动操作。
                if (mStackSupervisor.attachApplicationLocked(app)) {
                    didSomething = true;
                }
            } catch (Exception e) {
               ...
            }
        }

        // 查看进程中是否要运行的Service
        if (!badApp) {
            try {
                didSomething |= mServices.attachApplicationLocked(app, processName);
            } catch (Exception e) {
                ...
            }
        }

        // 查看进程中是否有要运行的Broadcast
        if (!badApp && isPendingBroadcastProcessLocked(pid)) {
            try {
                didSomething |= sendPendingBroadcastsLocked(app);
            } catch (Exception e) {
                ...
            }
        }
        ...
        return true;
    }
{% endcodeblock%}
attachApplicationLocked中做了两个重要的事情

1）bindApplication：完成Application的实例化操作。通过Binder机制调用ApplicationThread的bindApplication，又会经过Handler发送Application绑定的操作，通过mInstrumentation来完成Application实例化，最后调用Application的onCreate()方法

2）attachApplicationLocked：接着步骤1）查找栈顶的Activity，如果存在MainActivity。则调用ActivityStackSupervisor的attachApplicationLocked方法执行启动Activity的任务。

{% codeblock lang:java ActivityStackSupervisor.java %}
    // 查找需要启动的Activity。
    boolean attachApplicationLocked(ProcessRecord app) throws RemoteException {
        final String processName = app.processName;
        boolean didSomething = false;
        // 对所有任务栈循环
        for (int displayNdx = mActivityDisplays.size() - 1; displayNdx >= 0; --displayNdx) {
            ArrayList<ActivityStack> stacks = mActivityDisplays.valueAt(displayNdx).mStacks;
            for (int stackNdx = stacks.size() - 1; stackNdx >= 0; --stackNdx) {
                // 
                final ActivityStack stack = stacks.get(stackNdx);
                if (!isFrontStack(stack)) {
                    continue;
                }
                // 如果是最前显示的栈，获取栈顶Activity的信息
                ActivityRecord hr = stack.topRunningActivityLocked(null);
                if (hr != null) {
                    if (hr.app == null && app.uid == hr.info.applicationInfo.uid
                            && processName.equals(hr.processName)) {
                        try {
                            // 启动栈顶Activity
                            if (realStartActivityLocked(hr, app, true, true)) {
                                didSomething = true;
                            }
                        } catch (RemoteException e) {
                            ...
                        }
                    }
                }
            }
        }
        return didSomething;
    }

    final boolean realStartActivityLocked(ActivityRecord r,
            ProcessRecord app, boolean andResume, boolean checkConfig)
            throws RemoteException {

        if (andResume) {
            ...
        }
        ...
        r.app = app;
        app.waitingToKill = null;
        r.launchCount++;
        r.lastLaunchTime = SystemClock.uptimeMillis();
        ...
        int idx = app.activities.indexOf(r);
        if (idx < 0) {
            app.activities.add(r);
        }
        ...
        final TaskRecord task = r.task;
        final ActivityStack stack = task.stack;
        try {
            ...
            List<ResultInfo> results = null;
            List<ReferrerIntent> newIntents = null;
            if (andResume) {
                results = r.results;
                newIntents = r.newIntents;
            }
            ...
            if (r.isHomeActivity() && r.isNotResolverActivity()) {
                // Home process is the root process of the task.
                mService.mHomeProcess = task.mActivities.get(0).app;
            }
            ...
            r.sleeping = false;
            r.forceNewConfig = false;
            mService.showAskCompatModeDialogLocked(r);
            r.compat = mService.compatibilityInfoForPackageLocked(r.info.applicationInfo);
            ProfilerInfo profilerInfo = null;
            if (mService.mProfileApp != null && mService.mProfileApp.equals(app.processName)) {
                ...
            }

            if (andResume) {
                app.hasShownUi = true;
                app.pendingUiClean = true;
            }
            ...
            // 启动Activity
            app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                    System.identityHashCode(r), r.info, new Configuration(mService.mConfiguration),
                    new Configuration(stack.mOverrideConfig), r.compat, r.launchedFromPackage,
                    task.voiceInteractor, app.repProcState, r.icicle, r.persistentState, results,
                    newIntents, !andResume, mService.isNextTransitionForward(), profilerInfo);
            ...

        } catch (RemoteException e) {
            ...
        }
        ...

        return true;
    }
{% endcodeblock %}

遍历所有Activity的任务找到最前显示的Activity的栈，取栈顶的Activity执行真正的Activity启动操作。同样需要Binder进行进程间通讯通知ApplicationThread执行scheduleLaunchActivity任务。

{% codeblock lang:java ActivityThread.java -> ApplicationThread.java %}

    private class ApplicationThread extends ApplicationThreadNative {
        @Override
        public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
                ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
                CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
                int procState, Bundle state, PersistableBundle persistentState,
                List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
                boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

            updateProcessState(procState, false);

            ActivityClientRecord r = new ActivityClientRecord();

            r.token = token;
            r.ident = ident;
            r.intent = intent;
            r.referrer = referrer;
            r.voiceInteractor = voiceInteractor;
            r.activityInfo = info;
            r.compatInfo = compatInfo;
            r.state = state;
            r.persistentState = persistentState;

            r.pendingResults = pendingResults;
            r.pendingIntents = pendingNewIntents;

            r.startsNotResumed = notResumed;
            r.isForward = isForward;

            r.profilerInfo = profilerInfo;

            r.overrideConfig = overrideConfig;
            updatePendingConfiguration(curConfig);
            // 发送启动的Activity的消息
            sendMessage(H.LAUNCH_ACTIVITY, r);
        }
    }

    private class H extends Handler {
        ...
        public void handleMessage(Message msg) {
            ...
            switch (msg.what) {
                ...
                case LAUNCH_ACTIVITY: {
                    ...
                    final ActivityClientRecord r = (ActivityClientRecord) msg.obj;
                    ...
                    handleLaunchActivity(r, null);
                    ...
                } break;
                ...
            }
            ...
        }   
        ...
    }
    private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        mSomeActivitiesChanged = true;
        ...
        // 启动Activity前初始化WindowManager的全局属性
        WindowManagerGlobal.initialize();
        // 1. 执行Activity的启动操作
        Activity a = performLaunchActivity(r, customIntent);
        // 启动成功
        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            // 2. 执行Activity的resume操作
            handleResumeActivity(r.token, false, r.isForward,
                    !r.activity.mFinished && !r.startsNotResumed);

            if (!r.activity.mFinished && r.startsNotResumed) {
                ...
                r.paused = true;
            }
        } else {
            // 出现异常时直接结束Activity
        }
    }
{% endcodeblock %}

先调用performLaunchActivity启动Activity，方法中执行Activity的onCreate()和start()方法。如果启动成功即返回的Activity不为null，则继续执行handleResumeActivity方法，方法中完成Activity调用onResume方法，完成整个Activity启动的过程。

{% codeblock lang:java ActivityThread%}
    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        ActivityInfo aInfo = r.activityInfo;
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);
        }
        // 收集启动的Activity信息。
        // 创建Activity的相关组件
        ComponentName component = r.intent.getComponent();
        if (component == null) {
            component = r.intent.resolveActivity(
                mInitialApplication.getPackageManager());
            r.intent.setComponent(component);
        }

        if (r.activityInfo.targetActivity != null) {
            component = new ComponentName(r.activityInfo.packageName,
                    r.activityInfo.targetActivity);
        }

        Activity activity = null;
        try {
            // 用ClassLoader加载MainActivity。实例化MainActivity
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
            ...
            r.intent.setExtrasClassLoader(cl);
            r.intent.prepareToEnterProcess();
            if (r.state != null) {
                r.state.setClassLoader(cl);
            }
        } catch (Exception e) {
            ...
        }

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            ...
            if (activity != null) {
                // 创建Activity上下文信息。
                Context appContext = createBaseContextForActivity(r, activity);
                // 
                CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
                Configuration config = new Configuration(mCompatConfiguration);
                ...

                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor);

                if (customIntent != null) {
                    activity.mIntent = customIntent;
                }
                r.lastNonConfigurationInstances = null;
                activity.mStartedActivity = false;
                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }

                activity.mCalled = false;
                // 调用callActivityOnCreate方法
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                ...
                r.activity = activity;
                r.stopped = true;
                if (!r.activity.mFinished) {
                    // 调用start方法
                    activity.performStart();
                    r.stopped = false;
                }
                if (!r.activity.mFinished) {
                    // 如果是异常恢复则调用onRestoreInstanceState方法来恢复状态
                    if (r.isPersistable()) {
                        if (r.state != null || r.persistentState != null) {
                            mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                                    r.persistentState);
                        }
                    } else if (r.state != null) {
                        mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
                    }
                }
                if (!r.activity.mFinished) {
                    activity.mCalled = false;
                    // 
                    if (r.isPersistable()) {
                        mInstrumentation.callActivityOnPostCreate(activity, r.state,
                                r.persistentState);
                    } else {
                        mInstrumentation.callActivityOnPostCreate(activity, r.state);
                    }
                    ...
                }
            }
            r.paused = true;
            // 绑定token
            mActivities.put(r.token, r);
        } catch (SuperNotCalledException e) {
            throw e;
        } catch (Exception e) {
            ...
        }

        return activity;
    }
{% endcodeblock %}
实例化Activity的组件信息，通过ClassLoader加载Activity，创建上下文信息附加到Activity中，然后由Instrumentation调用callActivityOnCreate等方法完成Activity启动时的生命周期方法，如onCreate()、onStart()等。
{% codeblock lang:java Instrumentation.java%}
    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        // 调用Activity的onCreate 
        activity.performCreate(icicle);
        postPerformCreate(activity);
    }
{% endcodeblock %}

到此，Android的应用程序启动与Activity相关的流程就完成了。
在整个流程中几个重要的类需要注意

- `ActivityManagerNative` ：是Binder的子类，是底层Binder驱动在Java类中的实现。因为是抽象类，所以具体实现由ActivityManagerService来完成。

- `ActivityManagerService `：简称AMS，是Android中最核心的服务，主要负责Android的四大组件的启动、切换、是调度和应用进程的管理和调度工作（如Activity的生命周期控制）。在系统启动的时候完成AMS的注册。

- `ActivityManagerProxy` ：是ActivityManagerService的远程代理，客户端调用ActivityManagerProxy的相关方法通过Binder机制实现IPC，完成与ActivityManagerService通信交互任务。

- `ActivityThread` ：Application的入口，从main方法开始创建应用相关的核心功能。如主线程的消息循环，Application初始化，绑定Application相关的服务等等，同时控制组件的生命周期操作。对应Application的主线程。

- `ApplicationtThreadNative` ：与`ActivityManagerNative`一样也是Binder子类。具体实现由ApplicationThread完成。

- `ApplicationThread`：完成AMS与ActivityThread之间的通信。

- `ApplicationThreadProxy` ：是ApplicationThread远程接口代理。负责与客户端ApplicationThread通讯


- `Instrumentation` ：每个应用绑定唯一的一个Instrumentation，每个Activity都一个对该对象的引用。ActivityThread通过Instrumentation来控制Activity的生命周期。

- `ActivityStackSupervisor`： ActivityStack的超级管理员。

- `ActivityStack` ：用于保存Activity的栈，决定是否要启动新的进程。

- `ActivityRecord ` ：用于Activity的信息存储，包括状态、进程名等。

- `TaskRecord ` ：Android中的Task的具体实现。记录ActivityRecord的任务栈。

### 参考链接

[Android Developer Activities](https://developer.android.com/guide/components/activities.html)

[Activity的启动方式和flag详解](http://blog.csdn.net/singwhatiwanna/article/details/9294285)

[Android 基础](http://blog.csdn.net/liuhe688/article/details/9494411)

[深入理解Java Binder和MessageQueue](http://blog.csdn.net/innost/article/details/47317823)

[Android应用程序启动过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6689748)

[深入理解Binder](http://blog.csdn.net/innost/article/details/47208049)

[【凯子哥带你学Framework】Activity启动过程全解析](http://www.jianshu.com/p/6037f6fda285)