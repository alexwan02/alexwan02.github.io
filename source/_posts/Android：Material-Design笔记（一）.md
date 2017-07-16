---
title: Android：Material Design笔记（一）
date: 2016-01-31 01:30:44
tags:
---
[Codelab for Android Design Support Library used in I/O Rewind Bangkok session](http://inthecheesefactory.com/blog/android-design-support-library-codelab/en)

[Material Design for Android](http://developer.android.com/design/material/index.html)

[Material Design Introduction](http://www.google.com/design/spec/material-design/introduction.html#)


### 一、准备
#### 1. style.xml配置
{% codeblock lang:xml res/style.xml %}
<!--NoActionBar -->
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>

{% endcodeblock %}
#### 2.gradle 依赖
在app的build.gradle中添加Design Support依赖
```java
compile 'com.android.support:support-v4:23.0.1'
compile 'com.android.support:appcompat-v7:23.0.1'
compile 'com.android.support:design:23.0.1'
```
#### 3.布局文件
{% codeblock lang:xml layout/activity_main.xml %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin">
</android.support.v4.widget.DrawerLayout>

{% endcodeblock %}

### 二、添加FAB
Floating Action Button (FAB) 是带有阴影效果的圆形Button。在`activity_main.xml`中添加FloatingActionButton布局，FAB需要有个父布局来确定它在布局文件中的具体位置，所以把FAB放在FrameLayout中。如：`android:layout_gravity="bottom|end"`表示FAB位于整个布局的右下角。

{% codeblock lang:xml layout/activity_main.xml %}
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.design.widget.FloatingActionButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:src="@drawable/ic_plus"
            app:fabSize="normal"/>
    </FrameLayout>
</android.support.v4.widget.DrawerLayout>

{% endcodeblock %}

#### 1. android:src
表示FAB中的图标。建议大小为40dp的透明png图片。
#### 2. app:fabSize
表示FAB的大小。有`normal`、`min`两个值。normal的大小为56dp，min的大小为40dp。

5.0的效果

<div>
<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot21.jpg" />
</div>

4.x的效果

<div>
<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot20.jpg" />
</div>

在同样的布局文件中，两个的margin并不一致。design library开发小组把这个问题已确认为bug。目前的解决方案是在api 21+的dimens资源中设置FAB的margin值设置为16dp，低于api 21的dimens资源中设置FAB的margin值为0dp

res/values/dimens.xml

{% codeblock lang:xml res/values/dimens.xml %}
<dimen name="codelab_fab_margin_right">0dp</dimen>
<dimen name="codelab_fab_margin_bottom">0dp</dimen>
{% endcodeblock %}

res/values-v21/dimens.xml

{% codeblock lang:xml res/values-v21/dimens.xml %}
<dimen name="codelab_fab_margin_right">16dp</dimen>
<dimen name="codelab_fab_margin_bottom">16dp</dimen>

{% endcodeblock %}

res/layout/activity_code_lab.xml 中设定FAB的margin值
{% codeblock lang:xml res/layout/activity_code_lab.xml %}
<android.support.design.widget.FloatingActionButton
    ...
    android:layout_marginBottom="@dimen/codelab_fab_margin_bottom"
    android:layout_marginRight="@dimen/codelab_fab_margin_right"
    .../>

{% endcodeblock %}
#### 3. app:elevation
FAB的阴影深度。默认为6dp
#### 4. app:pressedTranslationZ
FAB按住时的阴影深度。默认为12dp
#### 5.app:backgroundTint
FAB背景颜色。默认使用样式中的`colorAccent`的颜色。
#### 6.FAB点击事件
```java
FloatingActionButton fabBtn;
...
fabBtn = (FloatingActionButton) findViewById(R.id.fabBtn);
    fabBtn.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
 
        }
    });
```
### 三、使用Snackbar
Snackbar，一个在屏幕底部显示的黑色条，可以设置显示的文本。类似Toast，与Toast不一样的是，Snackbar为UI的一部分，而Toast是浮动在屏幕上。

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/snackbar.jpg">
</div>

#### 1. 实例化 Snackbar
Snackbar make(@NonNull View view, @NonNull CharSequence text, int duration)

- view Snackbar 的显示位置
- text 显示的文字
- duration  Snackbar显示的时长 `LENGTH_SHORT`或`LENGTH_LONG`
```java
FrameLayout rootLayout;
...
rootLayout = (FrameLayout) findViewById(R.id.rootLayout);
...
fabBtn = (FloatingActionButton) findViewById(R.id.fabBtn);
fabBtn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Snackbar.make(rootLayout, "Hello. I am Snackbar!", Snackbar.LENGTH_SHORT)
                .setAction("Undo", new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {

                    }
                })
                .show();
    }
});
```
效果：

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot24.jpg">
</div>

Snackbar显示的有问题，是因为Snackbar需要Design Library的CoordinatorLayout布局配合来使用。
### 四、CoordinatorLayout
CoordinatorLayout是由Design Library的新添的布局，可以让包含的View和相协调。必须保证每个子View是为配合CoordinatorLayout的布局而设计和实现的。其中FAB和Snackbar就是其中两个。
#### 替换`FrameLayout`。

{% codeblock lang:xml res/activity/main_activity.xml %}
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/rootLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <android.support.design.widget.FloatingActionButton
        ... />
</android.support.design.widget.CoordinatorLayout>

{% endcodeblock %}

同时修改Activity中rootLayout的类型为CoordinatorLayout。

```java
//FrameLayout rootLayout;
CoordinatorLayout rootLayout;
...
//rootLayout = (FrameLayout) findViewById(R.id.rootLayout);
rootLayout = (CoordinatorLayout) findViewById(R.id.rootLayout);
```
效果图：

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot19n.png">
</div>

在android api 4.x上会有显示的bug。FAB的margin变为0dp。调整dimens的值

{% codeblock lang:xml res/values/dimens.xml %}
<dimen name="codelab_fab_margin_right">16dp</dimen>
<dimen name="codelab_fab_margin_bottom">16dp</dimen>

{% endcodeblock %}

修改完毕

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot20n.png">
</div>

`注` ：如果使用`Android Design Support `库的话，考虑使用`CoordinatorLayout`作为布局根view。

### 五、Toolbar
相比传统的ActionBar，Toolbar更加灵活。
#### 1.隐藏ActionBar
{% codeblock lang:xml res/values/styles.xml %}
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="windowActionBar">false</item>
    <item name="windowNoTitle">true</item>
</style>

{% endcodeblock %}

#### 2.添加Toolbar
（1）、在布局main_activity.xml添加Toolbar

{% codeblock lang:xml res/layout/main_activity.xml %}
<android.support.design.widget.CoordinatorLayout
    ...>
    <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"/>
        </android.support.design.widget.AppBarLayout>
    </android.support.design.widget.FloatingActionButton>
</android.support.design.widget.CoordinatorLayout>

{% endcodeblock %}

（2）、在Activity中调用setSupportActionBar为Activity指定toolBar。

{% codeblock lang:java MainActivity%}
public class MainActivity extends AppCompatActivity {
    Toolbar toolBar;
    DrawerLayout drawerLayout;
    ActionBarDrawerToggle drawerToggle;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        ActionBar actionBar = getSupportActionBar();
        if(actionBar != null){
            // 是否显示icon
            actionBar.setDisplayShowHomeEnabled(true);
            actionBar.setDisplayHomeAsUpEnabled(true);
            // 是否显示标题
            actionBar.setDisplayShowTitleEnabled(true);
        }
        drawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        drawerToggle = new ActionBarDrawerToggle(this , drawerLayout , 0 , 0);
        ...
    }
    ...
    @Override
    protected void onPostCreate(@Nullable Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        // toolbar 导航图标样式为home
        drawerToggle.syncState();
    }
    ...
}

{% endcodeblock %}

### 六、内容布局
我们介绍了FAB、Toolbar，现在为`main_activity.xml`布局主区域添加一些内容。如：添加带有两个button的LinearLayout。
{% codeblock lang:xml res/layout/main_activity.xml%}
<android.support.design.widget.AppBarLayout>
...
</android.support.design.widget.AppBarLayout>
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Yo Yo"
        />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Yo Yo"
        />
</LinearLayout>
<android.support.design.widget.FloatingActionButton
    ...>
{% endcodeblock%}

如下图

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot29.jpg">
</div>

发现两个buttom并不是我们想要的样式：置于toolbar下面，而是被toolbar遮住。因为`LinearLayout`并不适配CoordinatorLayout，所以没有像toolbar那样直接自适应CoordinatorLayout的内容。解决这个问题也很简单，在`LinearLayout`中加上`app:layout_behavior="@string/appbar_scrolling_view_behavior"`这个属性即可完美适配CoordinatorLayout布局了。
```java
<LinearLayout
    ...
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    ...
    >
```

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot30.jpg">
</div>

内容介绍结束。
### 七、TabLayout
Material Design提供新的tab布局TabLayout，在这之前实现这个功能一般选择第三方库`SlidingTabLayout`或`SlidingTabStrip `。引入TabLayout与toolbar类似需要在xml布局文件中添加TabLayout的布局，并在Java代码中设定TabLayout的Tab属性。

#### 1.main_activity.xml

{% codeblock lang:xml res/layout/main_activity.xml%}
...
<android.support.design.widget.AppBarLayout ...>
 
    <android.support.v7.widget.Toolbar ... />
 
    <android.support.design.widget.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
 
</android.support.design.widget.AppBarLayout>
...

{% endcodeblock %}

#### 2.在Java代码中添加Tab

{% codeblock lang:java MainActivity.java%}
TabLayout tabLayout;
...
tabLayout = (TabLayout) findViewById(R.id.tabLayout);
tabLayout.addTab(tabLayout.newTab().setText("Tab 1"));
tabLayout.addTab(tabLayout.newTab().setText("Tab 2"));
tabLayout.addTab(tabLayout.newTab().setText("Tab 3"));

{% endcodeblock %}

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot31.jpg">
</div>

#### 3.修改Tab的字体颜色
Tab的字体颜色是默认为黑色，用`app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"`可以修改Tab的字体颜色为白色

{% codeblock lang:xml res/layout/main_activity.xml %}
...
<android.support.design.widget.TabLayout
    ...
    app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar" />
...

{% endcodeblock %}

<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/screenshot32.jpg">
</div>

#### 4.适配ViewPager
调用`setupWithViewPager()` 方法为TabLayout指定ViewPager容器。
#### 5. app:tabMode 与 app:tabGravity属性
app:tabMode 
设置为`fixed`时，显示所有的Tab。适应于少量的Tab。不确定Tab的数量或Tab的数量较多的情况时，使用`scrollable `属性值，可以像Google Play Store那样滚动Tab。

app:tabGravity 设置为`fill `时，Tab显示的方式为填充，设置为`center `时，则居中显示Tab。
>如果app:tabMode 的属性值为`scrollable`时，app:tabGravity的属性值则被忽略。


<div>
	<image src="http://inthecheesefactory.com/uploads/source/designlibrary/tabmodetabgravity.jpg" style="width:100%">
</div>

### 八、让AppBarLayout随内容滚动退出屏幕
Android UX中设计非常好的一点就是可以让AppBarLayout随内容滚动退出屏幕，可以增加内容区域更多的显示空间。
#### 1.修改布局文件
使用NestedScrollView替换LinearLayout。
{% codeblock lang:xml res/layout/main_activity.xml %}
<android.support.v4.widget.NestedScrollView ...>
    <LinearLayout ...>
        ...
    </LinearLayout>
</android.support.v4.widget.NestedScrollView>

{% endcodeblock %}

#### 2.设置Toolbar的Scroll Flags属性值
{% codeblock lang:xml res/layout/main_activity.xml %}
...
<android.support.v7.widget.Toolbar
    ...
    app:layout_scrollFlags="scroll|enterAlways" />
...

{% endcodeblock %}
