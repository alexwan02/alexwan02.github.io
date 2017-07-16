title: Android：DataBinding Guide
date: 2016-07-17 16:56:24
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/c_scale,w_500/v1468746215/android/android_data_binding.png
tags: android-mvvm
categories: android-mvvm
---
## 一、编译环境配置
### 1. build.gradle
```java
android{
    ...
    dataBinding{
        enabled true
    }
}
```
如果在依赖module中需要支持databinding库，则在对应的依赖module的build.gradle配置文件添加dataBinding。
### 2. DataBinding 布局文件
使用DataBinding库的布局文件与普通布局文件不同的是：跟标签使用`<layout/>`：包含`<data></data>`标签和普通的视图布局。
{% codeblock lang:xml activity_main.xml %}
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
{% endcodeblock %}

示例中的<data>中的user变量用<variable>标签来引用：用`@{}`语法表示绑定的参数
```xml
...
<data>
    ...
    <variable name="user" type="com.example.User"/>
    ...
</data>
...
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>

```
`@{user.firstName}`意味：User中的firstName对应的具体
Java POJO 类有两种方式
1. 老式；一旦数据初始化后，值无法再次修改
	```java
	public class User{
		public final String firstName;
		public final String lastName;
		public User(String firstName , String lastName){
		    this.firstName = firstName;
		    this.lastName = lastName;
		}
	}
	```
2. 或使用私有private的java POJO类
	```java
	public class User {
	   private final String firstName;
	   private final String lastName;
	   public User(String firstName, String lastName) {
	       this.firstName = firstName;
	       this.lastName = lastName;
	   }
	   public String getFirstName() {
	       return this.firstName;
	   }
	   public String getLastName() {
	       return this.lastName;
	   }
	}
	```
从DataBinding角度来看，两个Java POJO类的效果相同。使用第一种Java类，TextView的android:text中表达式`@{user.firstName}`直接访问User的firstName字段。如果使用第二种类型Java类，则通过调用`getFirstName()`来访问User的firstName字段。
> 注：如果User类中存在firstName()方法，则首先调用firstName()方法。

### 3. 绑定数据
默认情况下，会根据布局文件的名称，在编译时自动生成首字母大写、驼峰式命名方式并以`Binding`结尾的Binding类。比如有名称为`main_activity.xml`的布局文件，则在编译时生成`MainActivityBinding.java`类。类中有布局绑定的data信息（如main_activity.xml中的user）并生成data的setter方法。最简单地创建binding方式就是在inflate布局时。

{% codeblock lang:java MainActivity.java %}
...
@override
protected void onCreate(Bundle savedInstanceState){
	super.onCreate(savedInstanceState);
	MainActivityBinding binding = DataBindingUtil.setContentView(this , R.layout.main_activity);
	User user = new User("Test" , "User");
	binding.setUser(user);
}
...
{% endcodeblock %}

或通过编译生成的MainActivityBinding类创建Binding

```java
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater);
```
如果绑定的是ListView或RecyclerView对应的Adapter，对应的binding类为ListItemBinding

```java
ListItemBinding binding = ListItemBinding.inflater(layoutInflater , viewGroup , false);
// or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

## 二、事件处理
DataBinding可以使用表达式来处理View的事件分发操作（如：onClick）。事件属性名由监听方法统一管理。如[View.onLongClickListener](https://developer.android.com/reference/android/view/View.OnLongClickListener.html)的方法[onLongClick()](https://developer.android.com/reference/android/view/View.OnLongClickListener.html#onLongClick(android.view.View))，则对应属性名为`android:onLongClick`。处理View事件分发有两种方式
1. 方法关联（[Method References](https://developer.android.com/topic/libraries/data-binding/index.html#method_references)）方式：在表达式中应用符合签名的监听方法。如果表达式签名与对应的方法引用一致，DataBinding将包裹引用的方法和对象到绑定的方法中。如果表达式为null，则不会创建监听。
2. 监听绑定（[Listener Bindings](https://developer.android.com/topic/libraries/data-binding/index.html#listener_bindings)）方式：在触发监听事件时，则调用绑定的lambda表达式。使用这个方式，DataBinding总是会创建View的监听事件。触发View事件时，调用lambda表达式绑定的方法。

### 1. 方法关联
直接将View对应的事件绑定到对应的方法。类似`android:onClick`关联Activity中的同名方法。比View#onClick属性相比，最大的好处就是这种方式只在编译时生成。所以在方法不存在或签名不正确时，则在编译阶段直接出错。
方法关联与监听绑定最大的不同就是：实际的Listener是在数据绑定时而不是在事件出发创建。如果希望只在View事件触发时执行表达式，则应该使用[监听绑定](https://developer.android.com/topic/libraries/data-binding/index.html#listener_bindings)的方式。
使用与处理方法名相同的表达式来关联监听事件，假设定义了如下的事件处理的类和方法
```java
public class MyHandlers{
    public void onClickFriend(View view){...}
}
```
则要在布局文件中通过<variable/> 来声明事件处理类

```xml
<?xml version="1.0" encoding="urf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <!-- 声明View的事件处理类 -->
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           <!-- 绑定声明的事件处理类的对应监听方；保证方法存在且命名相同 -->
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```
>注：要保证表达式中的方法名签名与事件处理类一致：方法存在且名称一致。

### 2. 监听绑定
与方法关联类似，但只在触发View响应事件时绑定表达式。表达式中允许存在多个参数（需要Gradle插件版本2.0以上）。
在方法关联中，方法参数必须符合事件监听的参数。在监听绑定中，只要求返回值符合监听表达式返回的值（除非返回void）。通过触发View事件调用绑定事件处理类中的方法。使用方法如下：
```java
public class Presenter{
	public void onSaveClick(Task task){}
}
```
绑定点击事件

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button 
            android:layout_width="wrap_content" 
            android:layout_height="wrap_content"
            android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>

```
使用lambda表达式的Listener只允许作为表达式的根元素。当表达式调用事件的回调方法时，DataBinding自动创建和注册必要的监听事件。当View分发事件时，DataBinding则通过事件分发调用绑定的表达式事件。在常规绑定的表达式中，可以得到值为null且线程安全的DataBinding，在执行View事件分发操作时。
在上面的例子中，我们并没有定义[onClick(android.view.View)](https://developer.android.com/reference/android/view/View.OnClickListener.html#onClick(android.view.View))方法中用到的view参数。在监听绑定的方式中，为Listener的参数提供了两种方式。

1. ）忽略监听中所有的方法或名称

	```java
	android:onClick="@{(view) -> presenter.onSaveClick(task)}"
	```
2. ）在自定义的方法中命名要使用的参数

	```java
	public class Presenter{
		public void onSaveClick(View view , Task task){};
	}
    // Layout 
	...
	android:onClick="@{(theView) -> presenter.onSaveClick(theView ,task)}"
	...
	```

可以使用有多个参数的lambda表达式
```java
public class Presenter {
    public void onCompletedChanged(Task task, boolean completed){}
}
// Layout

<CheckBox android:layout_width="wrap_content" 
    android:layout_height="wrap_content"
    android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

如果监听的事件返回值不是`void`类型，则自定义的表达式也必须要返回相同类型的值。如监听`onLongClick`时，则自定义的表达式必须要返回boolean类型。
```java
public class Presenter {
    public boolean onLongClick(View view, Task task){}
}
// Layout
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

如果因为null对象无法执行表达式，则Data Binding 会返回默认同种类型的Java 值。如引用类型为null，int为0，boolean默认值false 等。
如果使用断言类型的表达式（如：三目运算），则可以使用`void`作为标识符
```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

### 3. 避免使用复杂监听
监听表达式可以让代码易读性更好。但是含有复杂表达式的监听只会让你的布局易读性变差，更难维护。这些表达式
应该尽量简单，在回调方法中来实现业务逻辑。如果有特殊点击事件，需要指定除"android:onClick"之外的属性，
来避免冲突。下面的属性用来避免重复

|        |         |         |
---- | ---- | ----- 
[SearchView](https://developer.android.com/reference/android/widget/SearchView.html)    | setOnSearchClickListener(View.OnClickListener)     | android:onSearchClick
[ZoomControls](https://developer.android.com/reference/android/widget/ZoomControls.html) | setOnZoomInClickListener(View.OnClickListener)    | android:onZoomIn
[ZoomControls](https://developer.android.com/reference/android/widget/ZoomControls.html) | setOnZoomOutClickListener(View.OnClickListener) | android:onZoomOut

## 三、布局详解
### 1. Imports
在data中不用或引入多个import元素。类似Java的import
```java
<data>
    <import type="android.view.View"/>
</data>
```
现在可以在表达式中使用View了
```xml
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```
导入的类名冲突时，可以使用别名来避免
```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```
导入类型可以用来作为变量和表达式的类型引用
```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
```
>注：Android Studio尚未支持import的处理，所以IDE可能无法处理导入的变量。但是应用仍可以正常运行。
可以在变量中使用完全限定名解决IDE的问题

```xml
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
表达式可以使用导入类型的静态字段和静态方法。

```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

### 2. Variables 变量
data中可以使用任意数量的variable元素
```java
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```
变量的类型检查均在编译时完成，所以如果是实现 [Observable](https://developer.android.com/reference/android/databinding/Observable.html)或 [Observable Collections](https://developer.android.com/topic/libraries/data-binding/index.html#observable_collections)类型的变量则对应声明时的类型。如果变量或基类没有实现Observable*系列接口，则无法被观察，即无法实现动态绑定。
当有不同配置的布局文件（如水平、垂直），变量则会被组合到一起，可能会出现命名冲突，所以这些布局的文件中不允许出现重复定义。
生成的绑定类会为每个变量生成getter和setter方法，在调用setter方法之前，变量为java默认值。在必要情况下，DataBinding会生成名为context的特殊变量，context的值为root view的[getContext](https://developer.android.com/reference/android/view/View.html#getContext())返回值。如果在布局中显示指定具有相同的名称的context, 默认值则被覆盖。
## 四、自定义生成Binding的类名
默认情况，Binding类根据布局文件名，以驼峰形式，以"Binding"为后缀的来命名，位于module中的databinding包下。如"contact_item.xml"会生成ContactItemBinding的Binding类。如果module包名为`com.example.my.app` ,则生成的Binding 类位于com.example.my.app.databinding包下。
Binding 类可以在data元素中用class 来重命名
```xml
<data class="ContactItem">
    ...
</data>
```
则生成ContactItem，位于module的databinding包中。
如果类需要生成在module的根目录，则添加添加前缀"."
```xml
<data class=".ContactItem">
    ...
</data>
```
或者指定包中
```xml
<data class="com.example.ContactItem">
    ...
</data>
```

### 3. Includes布局
如果使用include方式添加布局文件时，可以用命名空间(namespace)和变量名传递
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```
>注：name.xml 和 contact中必须也有user变量
对应的include的布局<data/>的变量名必须要与引用它的布局属性名一致

Data Binding不支持使用<merge>作为直接子元素的布局，例如不支持类似以下的使用方式
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```
### 4. 支持的表达式运算符

1）常用特性
与Java表达式类似

- 四则运算      + - / * %
- 字符串连接   +
- 逻辑运算符   && ||
- 二元运算符   & | ^
- 一元运算符   + - ! ~
- 移位运算符   >> >>> <<
- 比较             == > < >= <=
- instanceof
- 组合 ()
- 迭代 - character, String, numeric, null
- 强制转换
- 方法调用
- 字段访问
- 数组调用
- 三目运算符 ?:

使用范例
```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age &lt; 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
2）不支持特性

- `不支持`Java 中 的this, super，new 操作符

3）空聚合操作符
```xml
android:text="@{user.displayName ?? user.lastName}"
```
与三目运算符`?:`相同
```xml
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```
4）属性引用
```xml
android:text="@{user.lastName}"
```
5）可以避免空指针
Data Binding自动检查null对象，避免空指针。如：@{user.name}, 如果user为空，则user.name为
默认值null。类似的user.age为0

6）集合
常用集合:arrays, lists, sparse lists, and maps。可以用[]操作符。
```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List<String>"/>
    <variable name="sparse" type="SparseArray<String>"/>
    <variable name="map" type="Map&lt<String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```
7）引用String值
如果属性使用单引号，则表达式使用双引号
```xml
android:text='@{map["firstName"]}'
```
如果属性使用双引号，String值使用单引号 或 双引号
```xml
android:text="@{map[`firstName`}"
android:text="@{map["firstName"]}"
```
8）引用资源文件
使用正常资源访问语法
```xml
<!-- 访问资源 -->
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```
根据String资源格式化字符串和复数。
```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

带有多个参数的复数，应该传递所有参数。
```xml
  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```
需要明确的一些资源类型

Type   |    Normal Reference | Expression Reference
------- | ------- | ---
String[]	|@array	|@stringArray
int[]	|@array	|@intArray
TypedArray	|@array	|@typedArray
Animator	|@animator	|@animator
StateListAnimator	|@animator	|@stateListAnimator
color int	|@color	|@color
ColorStateList	|@color	|@colorStateList

## 五、Data对象
旧式Java POJO可以用做数据绑定，但是修改数据时不能更新UI。而真正的Data Binding在数据变化时能够来修改。
有三种不同的数据变化通机制。[Observable objects](https://developer.android.com/topic/libraries/data-binding/index.html#observable_objects)、[observable fields](https://developer.android.com/topic/libraries/data-binding/index.html#observablefields)、[observable collection](https://developer.android.com/topic/libraries/data-binding/index.html#observable_collections)。当这些可观察的数据对象绑定UI后，在data对象属性值变化时，会自动更新UI。
### 1. 可观察对象（Observable Object）
实现[Observable](https://developer.android.com/reference/android/databinding/Observable.html)接口的类可以绑定一个单独的Listener来监听对象属性值的变化。[Observable](https://developer.android.com/reference/android/databinding/Observable.html)接口提供了添加和移除监听的机制。[BaseObservable](https://developer.android.com/reference/android/databinding/BaseObservable.html)就是实现了Observable的监听注册机制的基类。为了能够让data数据对象在属性值变化时发出通知，可以通过为属性值的getter方法设定[Bindable](https://developer.android.com/reference/android/databinding/Bindable.html)注解并在setter方法中添加通知。
```java
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}

```
在编译时[Bindable](https://developer.android.com/reference/android/databinding/Bindable.html)注解会在BR类中生成对应的访问入口。BR类会在编译时自动在module中生成。如果
无法修改data类的基类，可以使用[PropertyChangeRegistry](https://developer.android.com/reference/android/databinding/PropertyChangeRegistry.html)实现[Observable](https://developer.android.com/reference/android/databinding/Observable.html)接口，更高效存储和通知监听。

### 2. 可观察字段（ObservableFields）
如果仅有少量属性值或为了节省时间可以使用[ObservableField](https://developer.android.com/reference/android/databinding/ObservableField.html)来创建[Observable](https://developer.android.com/reference/android/databinding/Observable.html)类。与ObservableField类似的还有[ObservableBoolean](https://developer.android.com/reference/android/databinding/ObservableBoolean.html), [ObservableByte](https://developer.android.com/reference/android/databinding/ObservableByte.html), [ObservableChar](https://developer.android.com/reference/android/databinding/ObservableChar.html), [ObservableShort](https://developer.android.com/reference/android/databinding/ObservableShort.html), [ObservableInt](https://developer.android.com/reference/android/databinding/ObservableInt.html), 
[ObservableLong](https://developer.android.com/reference/android/databinding/ObservableLong.html), [ObservableFloat](https://developer.android.com/reference/android/databinding/ObservableFloat.html), [ObservableDouble](https://developer.android.com/reference/android/databinding/ObservableDouble.html)和[ObservableParcelable](https://developer.android.com/reference/android/databinding/ObservableParcelable.html)。ObservableFields是独立的可观察对象。使用基本数据类型可以避免封箱和拆箱操作。使用时需要在数据类中创建`public final`属性的字段
```java
private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```
使用set和get方法来存取属性值。
```java
user.firstName.set("Google");
int age = user.age.get();
```
### 3. 可观察集合（Observable Collections）
一些应用需要使用动态接口（如Map）来保存数据。Observable Collections可以通过键（key）来存取集合中的对象。如在键值为引用类型时，可以使用[ObservableArrayMap](https://developer.android.com/reference/android/databinding/ObservableArrayMap.html)。
```java
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);
```
在布局文件中 可以通过key来访问map中对象。

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
如果使用整型作为键值时，可以考虑使用[ObservableArrayList](https://developer.android.com/reference/android/databinding/ObservableArrayList.html)
```java
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```
在布局中使用索引来访问list集合
```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
   android:text='@{user[Fields.LAST_NAME]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

## 六、生成的Binding类
生成的Binding能够通过layout中的Views对象连接布局变量。如上所述，Binding类的命名和包名可以[自定义](https://developer.android.com/topic/libraries/data-binding/index.html#custom_binding_class_names)，生成的Binding类都继承自[ViewDataBinding](https://developer.android.com/reference/android/databinding/ViewDataBinding.html)。

### 1. 创建Binding类对象
Binding类对象应该在inflate视图之后尽快创建，保证在表达式绑定View时，View Hierarchy不会被干扰。绑定布局文件有不同方式，最常用是用Binding类的静态方法。
```java
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflate);
// 或者
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflate , viewGroup , false);
```
如果布局使用其他方式inflate，则需要分开来完成绑定。
```java
MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
```
有时不能预先知道binding。在这些情况可以使用[DataBindingUtil](https://developer.android.com/reference/android/databinding/DataBindingUtil.html)类
```java
ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
    parent, attachToParent);
ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
```
### 2. 使用ID的View
当为View指定一个ID时，则会在Binding类中为View生成一个`public final`属性的字段，直接获取View，比通过调用findViewById更加快速。如
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```
则会在Binding类中生成如下的字段
```java
public final TextView firstName;
public final TextView lastName;
```
对于data binding来说 ID几乎没有必要，但是在需要在代码中来获取view实例时仍有必要。
### 3. Variables变量
会为布局中每个声明的变量生成getter和setter方法的。
```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```
会在Binding类中生成如下方法
```java
public abstract com.example.User getUser();
public abstract void setUser(com.example.User user);
public abstract Drawable getImage();
public abstract void setImage(Drawable image);
public abstract String getNote();
public abstract void setNote(String note);
```
### 4. ViewStubs布局
[ViewStub](https://developer.android.com/reference/android/view/ViewStub.html)与常用View有些不同：起始时不可见，
```xml
 <ViewStub android:id="@+id/stub"
     android:inflatedId="@+id/subTree"
     android:layout="@layout/mySubTree"
     android:layout_width="120dip"
     android:layout_height="40dip" />
```
用setVisiable变为可见或显式调用inflate时，则用android:layout中的定义布局替换自己与`include`类似。但与`include`不同的的是在初始时并不会inflate到布局文件中，所以Binding的view不会在初始化时完成绑定。因为Binding中的View为final属性，使用[ViewStubProxy](https://developer.android.com/reference/android/databinding/ViewStubProxy.html)代理对象来替代ViewStud，可以在ViewStud创建后来访问。同样在ViewStud被inflate时，能够访问inflated的View Hierarchy。当inflate ViewStud的布局时，必须为新布局建立Binding。因此ViewStubProxy必须监听[ViewStub.OnInflateListener](https://developer.android.com/reference/android/view/ViewStub.OnInflateListener.html)并在inflate时建立Binding。又因为只能有一个存在，ViewStubProxy在创建Binding类对象后调用OnInflateListener。

## 七、高级Binding
### 1. 动态变量
有时不知道指定的Binding类。如： 操作不同布局的[RecyclerView.Adapter](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html)不知道指定的Binding类，但必须要在[onBindViewHolder(VH , int)](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html#onBindViewHolder(VH, int))
中关联Binding值。
例子中所有RecyclerView绑定的布局都有一个item变量。BindingHolder有一个getBinding方法返回[ViewDataBinding](https://developer.android.com/reference/android/databinding/ViewDataBinding.html)基类。
```java
public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}
```
### 2. 立即绑定
当变量或观察者变化时，Binding在下一帧前执行改变操作。有时Binding必须立即执行。强制执行使用 [executePendingBindings()](https://developer.android.com/reference/android/databinding/ViewDataBinding.html#executePendingBindings())方法
### 3. 后台线程
DataBinding可以在后台线程中修改非集合类型的data数据，在线程中执行时，DataBinding会本地化变量和字段，避免多线程引起的同步问题。

## 七、属性Setter
无论何时绑定值变化，生成的binding类都必须view中绑定的表达式setter方法。DataBinding框架能够自定义setter的方法。
### 1. 自动setter
对于属性attribute来说，DataBinding会尝试查找setAttribute方法。attribute的命名空间可以或略，只针对属性的名称。如：关联TextView的android:text属性表达式会查找setText方法。如果表达式返回int，DataBinding会搜索setText(int)的方法。注意要用返回正确类型的表达式，如果必要的话，使用强制类型转换。
>注：即使不存在指定的属性名，DataBinding也会工作。

你可以使用DataBinding为任意setter方法创建对应的属性名。如：依赖库DrawerLayout没有任何属性名，但是仍有很多setter，可以使用自动的属性名setter方法
```xml
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}"/>
```
### 2. setter重命名
有点setter方法没有对应的属性名。对于这些方法，可以用[BindingMethods](https://developer.android.com/reference/android/databinding/BindingMethods.html)注解关联指定的属性名，使用这种方式时必须关联用[BindingMethods](https://developer.android.com/reference/android/databinding/BindingMethod.html)注解的类。如`android:tint`属性名实际关联的是[setImageTintList(ColorStateList)](https://developer.android.com/reference/android/widget/ImageView.html#setImageTintList(android.content.res.ColorStateList))方法而不是`setTint()`。
```java
@BindingMethods({
       @BindingMethod(type = "android.widget.ImageView",
                      attribute = "android:tint",
                      method = "setImageTintList"),
})
```
一般来说没有必要重命名setters，因为android框架属性已经实现。
### 3. 自定义setter
有时需要自定义属性的绑定逻辑。如：`android:paddingLeft`没有对应的setter实现方法，但是存在`setPadding(left, top, right, bottom) `方法。使用` BindingAdapter`注解的静态方法可以用来自定义属性名的setter逻辑。
android属性已经有创建的[BindingAdapter](https://developer.android.com/reference/android/databinding/BindingAdapter.html)，如： paddingLeft
```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int padding) {
   view.setPadding(padding,
                   view.getPaddingTop(),
                   view.getPaddingRight(),
                   view.getPaddingBottom());
}
```
BindingAdapter对自定义类型的属性很有用。如：创建图片加载方法。
>注：当默认的Adapter与自定义的BindingAdapter冲突时，自定义的BindingAdapter覆盖默认值。

```java
// 1. 全部满足
@BindingAdapter({"bind:imageUrl", "bind:error"})
public static void loadImage(ImageView view, String url, Drawable error) {
   Picasso.with(view.getContext()).load(url).error(error).into(view);
}
// 2. 满足其中一个时
@BindingAdapter(value = {"bind:imageUrl", "bind:error"} , requireAll = false)
public static void loadImage(ImageView view, String url, Drawable error) {
   Picasso.with(view.getContext()).load(url).error(error).into(view);
}

// Layout
<ImageView app:imageUrl="@{venue.imageUrl}"
app:error="@{@drawable/venueError}"/>
```
使用第一种BindingAdapter时，必须要同时声明`bind:imageUrl`和`bind:error`两个属性时，才会调用。

- 在属性名匹配BindingAdapter时，自动会忽略属性名之前的命名空间
- 可以自定义以android为命名空间的属性值（此时自定义覆盖默认）。

BindingAdapter可以用在处理方法中使用旧值。如：在新值与旧值不同时，使用新值。
```java
@BindingAdapter("android:paddingLeft")
public static void setPaddingLeft(View view, int oldPadding, int newPadding) {
   if (oldPadding != newPadding) {
       view.setPadding(newPadding,
                       view.getPaddingTop(),
                       view.getPaddingRight(),
                       view.getPaddingBottom());
   }
}
```
事件处理器可以使用接口类或带有一个抽象方法的抽象类作为参数。如
```java
@BindingAdapter("android:onLayoutChange")
public static void setOnLayoutChangeListener(View view, View.OnLayoutChangeListener oldValue,
       View.OnLayoutChangeListener newValue) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        if (oldValue != null) {
            view.removeOnLayoutChangeListener(oldValue);
        }
        if (newValue != null) {
            view.addOnLayoutChangeListener(newValue);
        }
    }
}
```
当Listener有多个方法时，必须拆分成多个Listener。如：[View.OnAttachStateChangeListener](https://developer.android.com/reference/android/view/View.OnAttachStateChangeListener.html)有两个方法
[onViewAttachedToWindow()](https://developer.android.com/reference/android/view/View.OnAttachStateChangeListener.html#onViewAttachedToWindow(android.view.View)) 和 [onViewDetachedFromWindow()](https://developer.android.com/reference/android/view/View.OnAttachStateChangeListener.html#onViewDetachedFromWindow(android.view.View))。我们必须创建两个接口来区分属性和处理方法。
```java
@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewDetachedFromWindow {
    void onViewDetachedFromWindow(View v);
}

@TargetApi(VERSION_CODES.HONEYCOMB_MR1)
public interface OnViewAttachedToWindow {
    void onViewAttachedToWindow(View v);
}
```
因为修改一个Listener可能会影响到其他的Listener，所以必须要创建三个不同的BindingAdapter
```java
@BindingAdapter("android:onViewAttachedToWindow")
public static void setListener(View view, OnViewAttachedToWindow attached) {
    setListener(view, null, attached);
}

@BindingAdapter("android:onViewDetachedFromWindow")
public static void setListener(View view, OnViewDetachedFromWindow detached) {
    setListener(view, detached, null);
}

@BindingAdapter({"android:onViewDetachedFromWindow", "android:onViewAttachedToWindow"})
public static void setListener(View view, final OnViewDetachedFromWindow detach,
        final OnViewAttachedToWindow attach) {
    if (VERSION.SDK_INT >= VERSION_CODES.HONEYCOMB_MR1) {
        final OnAttachStateChangeListener newListener;
        if (detach == null && attach == null) {
            newListener = null;
        } else {
            newListener = new OnAttachStateChangeListener() {
                @Override
                public void onViewAttachedToWindow(View v) {
                    if (attach != null) {
                        attach.onViewAttachedToWindow(v);
                    }
                }

                @Override
                public void onViewDetachedFromWindow(View v) {
                    if (detach != null) {
                        detach.onViewDetachedFromWindow(v);
                    }
                }
            };
        }
        final OnAttachStateChangeListener oldListener = ListenerUtil.trackListener(view,
                newListener, R.id.onAttachStateChangeListener);
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener);
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener);
        }
    }
}
```
上面的例子稍微复杂一些，因为View使用添加和移除Listener方式，而不是为[View.OnAttachStateChangeListener](https://developer.android.com/reference/android/view/View.OnAttachStateChangeListener.html)设置
setter方法。android.databinding.adapters.ListenerUtil类帮助跟踪之前的Listener，这样他们可以在
Binding适配器中移除旧的Listener。
通过 @TargetApi(VERSION_CODES.HONEYCOMB_MR1)注解接口OnViewDetachedFromWindow和OnViewAttachedToWindow，DataBinding代码生成器就会知道只有运行 Honeycomb MR1版本及以上的新设备才生成Listener。
## 八、转换器Converters
### 1. 对象转换
如果是表达式返回的对象，则会从自动的、重命名或自定义的setter方法中选择一个setter方法。对象会强转成适合setter方法中参数类型。这种方式适合用ObservableMaps来保存数据的布局，如：
```xml
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
`userMap`返回的对象自动强转为`setText(CharSequence)`中参数类型。当setter方法的参数类型存在二义性时，如存在两个setter方法，但是参数不一样时，需要使用手动强转。
### 2. 自定义转换 
有时需要让布局在指定类型之前自动转换，如设置背景时
```xml
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
这里android:background的setter方法参数类型为`Drawable `，但是Color是整型。可以将整型的color转为Drawable类型的`ColorDrawable `，通过BindingConversion注解的静态方法来实现。
```java
@BindingConversion
public static ColorDrawable convertColorToDrawable(int color) {
   return new ColorDrawable(color);
}
```
>注：转换器只在setter层执行，禁止使用混合类型。如：
```xml
<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```
## 九、Android Studio对Data Binding的支持
 Android Studio支持很多data binding代码的编辑特性。如
 - 语法高亮
 - 标记表达式语法错误。
 - XML代码补全
 - 引用包括导航和快速文档

>注：Arrays和[生成类型](https://docs.oracle.com/javase/tutorial/java/generics/types.html)如[Observable](https://developer.android.com/reference/android/databinding/Observable.html)类，可能会出现显示错误（实际上没有错误）
 预览窗口显示data binding的默认值。如：
```xml
 <TextView android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="@{user.firstName, default=PLACEHOLDER}"/>
```
 预览窗口会显示PLACEHOLDER作为TextView默认值。
 如果需要在项目设计时显示默认值，可以使用tools属性替代默认的表达式。具体见[Designtime Layout Attributes](http://tools.android.com/tips/layout-designtime-attributes)

## 参考
[Data Binding Library](https://developer.android.com/topic/libraries/data-binding/index.html)

[棉花糖给 Android 带来的 Data Bindings](https://realm.io/cn/news/data-binding-android-boyar-mount/)

[2-way Data Binding on Android](https://halfthought.wordpress.com/2016/03/23/2-way-data-binding-on-android/)
