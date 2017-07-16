---
title: Android：RecyclerView
date: 2016-02-06 06:47:38
tags:
---
[RecyclerView basics](http://enoent.fr/blog/2015/01/18/recyclerview-basics/)
### 一、RecylerView 介绍
`RecyclerView`是Android5.0之后V7包中的新特性。与`ListView`相似，但是比`ListView`更灵活，支持Android 2.1版本以上。
正如它的名字：当一个item隐藏的时候，隐藏的item会被再回收重用，绑定新的数据。而不是被销毁为新的item的创建新的布局。
RecylerView分为6个主要组件

1. `Adapter`：提供数据，类似`ListView`的
2. `ItemAnimator`：item的添加、删除、修改、移动的动画效果
3. `ItemDecoration`：添加图画或修改Item布局
4. `LayoutManager`：指定Item如何布局（List、Grid...）
5. `ViewHolder`：每个Item View的基础类
6. `RecylerView`：将所有的组件绑定

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548610/android/recyclerview_uml.png" style="width:80%;">
</div>

### 二、RecyclerView展示
#### 1.准备
Gradle添加 `RecyclerView `的supprt-v7包依赖
```java
compile 'com.android.support:recyclerview-v7:21.0.3'
```
本篇文章也使用`CardViews`，因此建议添加对应的依赖
```java
compile 'com.android.support:cardview-v7:21.0.3'
```
#### 2.BaseItem
`BaseItem` 包含一个`title` 和一个`subtitle `属性。用来表示列表中每个`item`
```java
public class Item{
  private String title;
  private String subtitle;
  Item(String title , String subtitle){
    this.title = title;
    this.subtitle = subtitle;
  }
  public String getTitle() {
    return title;
  }

  public String getSubtitle() {
    return subtitle;
  }
}
```
#### 3.Item Layout
使用`CardView`作为Item的展示Layout。`CardView`是具有装饰的`FrameLayout `。
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  android:layout_width="match_parent
  android:layout_height="match_parent"
  app:contentPadding="8dp"
  app:cardUseCompatPadding="true" >

  <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical" >

      <TextView
          android:id="@+id/title"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:singleLine="true"
          style="@style/Base.TextAppearance.AppCompat.Headline" />

      <TextView
          android:id="@+id/subtitle"
          android:layout_width="match_parent"
          android:layout_height="0dp"
          android:layout_weight="1"
          style="@style/Base.TextAppearance.AppCompat.Subhead" />

  </LinearLayout>

</android.support.v7.widget.CardView>
```
#### 4.adapter

1. 定义`ViewHolder`类，必须继承`RecyclerView.ViewHolder`。同时需要存储绑定数据时需要使用到的View的对象。
例子中我们需要两个TextView
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  @SuppressWarnings("unused")
  private static final String TAG = Adapter.class.getSimpleName();

  public static class ViewHolder extends RecyclerView.ViewHolder {
    TextView title;
    TextView subtitle;

    public ViewHolder(View itemView) {
        super(itemView);

        title = (TextView) itemView.findViewById(R.id.title);
        subtitle = (TextView) itemView.findViewById(R.id.subtitle);
    }
  }
}
```
2. 使用`Collection`存储对象的集合，简单起见，例子中使用`ArrayList`来存储列表数据。
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  @SuppressWarnings("unused")
  private static final String TAG = Adapter.class.getSimpleName();
  
  private static final int ITEM_COUNT = 50;
  private List<Item> items;

  public Adapter() {
    super();

    // 创建item的列表
    items = new ArrayList<>();
    for (int i = 0; i < ITEM_COUNT; ++i) {
        items.add(new Item("Item " + i, "This is the item number " + i));
    }
  }
  // 省略ViewHolder的定义
}
```
3. 实现`RecyclerView.Adapter`的方法。
- onCreateViewHolder(ViewGroup parent, int viewType) 

	创建View，返回匹配的ViewHolder
- onBindViewHolder(ViewHolder holder, int position) 
	
	根据Item的position，给ViewHolder绑定数据
- getItemCount()

	返回Adapter的item的数量
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  // Attributes and constructor omitted

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item, parent, false);
    return new ViewHolder(v);
  }

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {
    final Item item = items.get(position);

    holder.title.setText(item.getTitle());
    holder.subtitle.setText(item.getSubtitle());
  }

  @Override
  public int getItemCount() {
    return items.size();
  }

 // 省略ViewHolder的定义
}
```
#### 5. Bind everything together

1. 在Activity的布局文件中添加`RecyclerView `
```java
<RelativeLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  tools:context=".MainActivity" >

  <android.support.v7.widget.RecyclerView
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />

</RelativeLayout>
```
2. 使用`LinearLayoutManager `，`DefaultItemAnimator `
```java
public class MainActivity extends ActionBarActivity {
  @SuppressWarnings("unused")
  private static final String TAG = MainActivity.class.getSimpleName();

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
    recyclerView.setAdapter(new Adapter());
    recyclerView.setItemAnimator(new DefaultItemAnimator());
    recyclerView.setLayoutManager(new LinearLayoutManager(this));
  }
}
```

3. 编译、运行

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548620/android/recyclerview-1.png" style="width:35%;">
</div>

### 三、相同RecyclerView显示不同类型的View
假设现在要显示两种类型的Item。如一个远程音乐的列表和一些离线的专辑。可以为他们指定行为，显示指定的信息。
1. 在Item中添加`active`属性
```java
public class Item {
  private String title;
  private String subtitle;
  private boolean active;

  Item(String title, String subtitle, boolean active) {
    this.title = title;
    this.subtitle = subtitle;
    this.active = active;
  }

  public String getTitle() {
    return title;
  }

  public String getSubtitle() {
    return subtitle;
  }

  public boolean isActive() {
    return active;
  }
}
```
2. 修改创建Item的方法，随机设定Item的`active`值，根据`active`来改变subtitle状态
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
    // 忽略参数

    public Adapter() {
        super();

        // 创建Item
        Random random = new Random();
        items = new ArrayList<>();
        for (int i = 0; i < ITEM_COUNT; ++i) {
            items.add(new Item("Item " + i, "This is the item number " + i, random.nextBoolean()));
        }
    }

    // 忽略onCreateViewHolder

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        final Item item = items.get(position);

        holder.title.setText(item.getTitle());
        holder.subtitle.setText(item.getSubtitle() + ", which is " +
                (item.isActive() ? "active" : "inactive"));
    }

    // …
}
```
3. 学会显示不同String是个好的开始，但是我们不仅仅要满足于此。在Adapter中你可能会注意到`onCreateViewHolder(ViewGroup parent, int viewType)`中的viewType我们没有使用。这里的`viewType`可以用来实现这样的需求：修改ViewHolder的实例。我们必须告诉`Adapter`来确定Item的类型。通过复写`getItemViewType(int position)`方法实现。
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  // …

  private static final int TYPE_INACTIVE = 0;
  private static final int TYPE_ACTIVE = 1;

  // …

  @Override
  public int getItemViewType(int position) {
    final Item item = items.get(position);

    return item.isActive() ? TYPE_ACTIVE : TYPE_INACTIVE;
  }

  // …
}
```
4. 现在根据你的需求有多种可能：为每种类型的View创建不同的`ViewHolder`或为相同的`ViewHolder` inflate 不同的Layout。简单起见，这里使用相同的`ViewHolder` inflate 不同的Layout。我们对`inactive `的Item继续使用现在的Layout，对`active`的Item创建一个新的Layout

{% codeblock lang:xml  layout/item_active.xml %}

<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:contentPadding="8dp"
    app:cardUseCompatPadding="true" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >

        <TextView
            android:id="@+id/title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:singleLine="true"
            android:textColor="@color/material_deep_teal_500"
            style="@style/Base.TextAppearance.AppCompat.Headline" />

        <TextView
            android:id="@+id/subtitle"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:textColor="@color/material_blue_grey_900"
            style="@style/Base.TextAppearance.AppCompat.Subhead" />

    </LinearLayout>
</android.support.v7.widget.CardView>

{% endcodeblock %}

5. 最后也是最重要：在`onCreateViewHolder`方法中根据`viewType`值Inflate不同的Layout

{% codeblock lang:java %}

public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  // …

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    final int layout = viewType == TYPE_INACTIVE ? R.layout.item : R.layout.item_active;

    View v = LayoutInflater.from(parent.getContext()).inflate(layout, parent, false);
    return new ViewHolder(v);
  }
  // …
}

{% endcodeblock %}

现在我们可以区分`active `和`inactive` Item了

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548620/android/recyclerview-active.png" style="width:35%;">
</div>

### 四、Layout managers
#### 1.LinearLayoutManager
这个也是上面我们用的Manager，复制了`ListView`的特性。`Layout managers`占用三个参数

- `Context` 不能为空  
- `int orientation ` vertical（默认值）、horizontal
- `boolean reverseLayout` 是否反转Layout

下面是`反转` `水平方向`的LinearLayoutManager

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548621/android/recyclerview-linear-horizontal-reversed.png" style="width:35%;">
</div>

设置反转Layout时显示最右边无需滑动

#### 2.GridLayoutManager
类似`GridView`。占用四个参数

- `Context ` 非空
- `int spanCount` 非空
- `int orientation` vertical（默认值）、horizontal
- `boolean reverseLayout` 是否反转Layout

下面是`spanCount`为3，`vertical`方向，`非反转`的GridLayoutManager

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548622/android/recyclerview-grid-3-vertical.png" style="width:35%;">
</div>

`注意`：GridLayoutManager的`reverseLayout`属性只反转设定的反向。如设置orientation为`vertical`时，只垂直反转，水平方向不变。

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548622/android/recyclerview-grid-3-vertical-reversed.png" style="width:35%;">
</div>

#### 3.StaggeredGridLayoutManager
StaggeredGridLayoutManager是`GridLayoutManager`的加强版。截止到`support-v7 21.0.0.3`版本的包中，StaggeredGridLayoutManager仍然还有很多[bug](https://code.google.com/p/android/issues/detail?id=93711)和[困扰的问题](https://code.google.com/p/android/issues/detail?id=93156)。和其他几种LayoutManager相比，`StaggeredGridLayoutManager`看上去像是匆忙完成的。同时需要注意的是它不需要`Context`，但是`orientation` 不能省略。同`GridLayoutManager`一样，需要指定`span`的数量。构造器中不提供`reverse` 参数，可以用代码来实现。
下面是固定Span的Grid，我们可以让Item占用整行或整列。让我们看看如何实现。用之前例子中创建的 `active/inactive`属性的item list，让`active`属性的item占满整行。在`adapter` 绑定数据的时候，来实现。
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
    // …
  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {
    final Item item = items.get(position);

    holder.title.setText(item.getTitle());
    holder.subtitle.setText(item.getSubtitle() + ", which is " + (item.isActive() ? "active" : "inactive"));

    // Span the item if active
    final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
    if (lp instanceof StaggeredGridLayoutManager.LayoutParams) {
        StaggeredGridLayoutManager.LayoutParams sglp = (StaggeredGridLayoutManager.LayoutParams) lp;
        sglp.setFullSpan(item.isActive());
        holder.itemView.setLayoutParams(sglp);
    }
  }

  // …
}
```
在代码判断如果LayoutManager是StaggeredGridLayoutManager的实例，来修改Layout参数。效果如图

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548621/android/recyclerview-staggered-grid.png" style="width:35%;">
</div>

### 五、Item点击响应
与ListView 不同的是RecyclerView没有提供相关的点击响应事件的API，但是依然很容易来实现。现在我们需要监听每个Item的`click`和`long-click`事件.每个Item是由`ViewHolder`来显示的，每个`ViewHolder`是由根`View` 来初始化的。OK，用`View`来作为`click`和`long-click`事件的回调。最后只需要把`ViewHolder`和它的position对应起来即可，`RecyclerView.ViewHolder`已经提供对应的方法~~getPosition()~~ ( getPosition() 方法废弃, 根据你的情况用`getLayoutPosition()` 或 `getAdapterPosition()`来替代 )来获取当前绑定的Item的postion
```java
public class Adapter extends RecyclerView.Adapter<Adapter.ViewHolder> {
  // …

  public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener,
        View.OnLongClickListener {
    @SuppressWarnings("unused")
    private static final String TAG = ViewHolder.class.getSimpleName();

    TextView title;
    TextView subtitle;

    public ViewHolder(View itemView) {
      super(itemView);

      title = (TextView) itemView.findViewById(R.id.title);
      subtitle = (TextView) itemView.findViewById(R.id.subtitle);

      itemView.setOnClickListener(this);
      itemView.setOnLongClickListener(this);
    }

    @Override
    public void onClick(View v) {
      Log.d(TAG, "Item clicked at position " + getPosition());
    }

    @Override
    public boolean onLongClick(View v) {
      Log.d(TAG, "Item long-clicked at position " + getPosition());
      return true;
    }
  }
}
```

### 六、选择处理
`long-click`事件一个常用的模式是触发选择模式。再一次，`RecyclerView`没有帮助我们实现，但是实现起来也很简单。
分为三步
- 维护一个选择状态
- 更新选中的Item的View
- 开启选择模式

为了说明，我们添加一个选择Item的方式，然后移除选中的Item。
#### 1.选择状态
我们需要修改`Adapter`来维护一个选中的item list。`Adapter`需要提供一下列表参数

- 选中的元素列表
- 改变给定元素的选择状态

我们加入额外的方法几个方法：

- 检查指定的元素是否被选中
- 清空选中
- 提供选中元素的数量

这里并不是随便选择这些方法，在下个部分我们需要它们。
我们注意到这五个方法都不是特定的Item，我们可以用普通方式写这些方法并重用`Adapter`的行为。
```java
public abstract class SelectableAdapter<VH extends RecyclerView.ViewHolder> extends RecyclerView.Adapter<VH> {
  @SuppressWarnings("unused")
  private static final String TAG = SelectableAdapter.class.getSimpleName();

  private SparseBooleanArray selectedItems;

  public SelectableAdapter() {
    selectedItems = new SparseBooleanArray();
  }

  /**
   * 指定对应位置的Item是否被选中
   * @param Item的position
   * @return 如果是选中返回true，否则返回false
   */
  public boolean isSelected(int position) {
    return getSelectedItems().contains(position);
  }

  /**
   * 给定postion的Item的选中状态触发器
   * @param position item 的 postion
   */
  public void toggleSelection(int position) {
    if (selectedItems.get(position, false)) {
      selectedItems.delete(position);
    } else {
      selectedItems.put(position, true);
    }
    notifyItemChanged(position);
  }

  /**
   * 清空所有Item的选中状态
   */
  public void clearSelection() {
    List<Integer> selection = getSelectedItems();
    selectedItems.clear();
    for (Integer i : selection) {
      notifyItemChanged(i);
    }
  }

  /**
   * 选中Item的数量
   * @return 选中Item的数量
   */
  public int getSelectedItemCount() {
    return selectedItems.size();
  }

  /**
   * 被选中的Item的Id list
   * @return 被选中的Item的Id list
   */
  public List<Integer> getSelectedItems() {
    List<Integer> items = new ArrayList<>(selectedItems.size());
    for (int i = 0; i < selectedItems.size(); ++i) {
      items.add(selectedItems.keyAt(i));
    }
    return items;
   }
}

```

最后`Adapter`继承我们创建`SelectableAdapter`
```java
public class Adapter extends SelectableAdapter<Adapter.ViewHolder> {
  // …
}
```
#### 2.更新Item的View
通知用户选中了一个Item，我们经常看到选中的View被覆盖某个颜色。在`item.xml`和`item_active.xml`中，我们添加一个`invisible`并带有颜色的View。因为这个View应该填充整个`CardView`，我们应该改变Layout的一些属性（将CardView的padding属性移动到LinearLayout中），View的颜色应该为透明的。
我们要用android Framework的`selectableItemBackground`为`CardView `的foreground设定属性值，来添加一个好看的触摸反馈。在Android 5.0以及以上，显示涟漪的效果，Android 5.0以下版本显示灰色。

{% codeblock lang:xml item.xml（item_active.xml做对应的修改）%}

<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:foreground="?android:attr/selectableItemBackground"
    app:cardUseCompatPadding="true" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="8dp" >

        <TextView
            android:id="@+id/title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:singleLine="true"
            style="@style/Base.TextAppearance.AppCompat.Headline" />

        <TextView
            android:id="@+id/subtitle"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            style="@style/Base.TextAppearance.AppCompat.Subhead" />

    </LinearLayout>

    <View
        android:id="@+id/selected_overlay"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/selected_overlay"
        android:visibility="invisible" />

</android.support.v7.widget.CardView>

{% endcodeblock %}

下一步决定什么时候显示这个效果，很明显需要 `Adapters`的`onBindViewHolder()`方法中。在`ViewHolder`中添加overlay view的属性

```java
public class Adapter extends SelectableAdapter<Adapter.ViewHolder> {
  // …

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {
    final Item item = items.get(position);

    holder.title.setText(item.getTitle());
    holder.subtitle.setText(item.getSubtitle() + ", which is " + (item.isActive() ? "active" : "inactive"));

    // Span the item if active
    final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
    if (lp instanceof StaggeredGridLayoutManager.LayoutParams) {
      StaggeredGridLayoutManager.LayoutParams sglp = (StaggeredGridLayoutManager.LayoutParams) lp;
      sglp.setFullSpan(item.isActive());
      holder.itemView.setLayoutParams(sglp);
    }

    //  高亮选中的Item
    holder.selectedOverlay.setVisibility(isSelected(position) ? View.VISIBLE : View.INVISIBLE);
  }

  // …

  public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener,
        View.OnLongClickListener {
    @SuppressWarnings("unused")
    private static final String TAG = ViewHolder.class.getSimpleName();

      TextView title;
      TextView subtitle;
      View selectedOverlay;

      public ViewHolder(View itemView) {
        super(itemView);

        title = (TextView) itemView.findViewById(R.id.title);
        subtitle = (TextView) itemView.findViewById(R.id.subtitle);
        selectedOverlay = itemView.findViewById(R.id.selected_overlay);

        itemView.setOnClickListener(this);
        itemView.setOnLongClickListener(this);
      }
      // …
  }
}
```

#### 3.开启选中模式
最后一步可能有点复杂，但也不是很难。我们需要将`click`和`long-click`返回给`Activity`。用`ViewHolder`暴露一个`Listener`，通过`Adapter`传递它，实现需求。
```java
public class Adapter extends SelectableAdapter<Adapter.ViewHolder> {
    // …

  private ViewHolder.ClickListener clickListener;

  public Adapter(ViewHolder.ClickListener clickListener) {
    super();

    this.clickListener = clickListener;

    // 创建Item
    Random random = new Random();
    items = new ArrayList<>();
    for (int i = 0; i < ITEM_COUNT; ++i) {
      items.add(new Item("Item " + i, "This is the item number " + i, random.nextBoolean()));
    }
  }

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    final int layout = viewType == TYPE_INACTIVE ? R.layout.item : R.layout.item_active;

    View v = LayoutInflater.from(parent.getContext()).inflate(layout, parent, false);
    return new ViewHolder(v, clickListener);
  }

  // …

  public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener,
          View.OnLongClickListener {
    @SuppressWarnings("unused")
    private static final String TAG = ViewHolder.class.getSimpleName();

    TextView title;
    TextView subtitle;
    View selectedOverlay;

    private ClickListener listener;

    public ViewHolder(View itemView, ClickListener listener) {
      super(itemView);

      title = (TextView) itemView.findViewById(R.id.title);
      subtitle = (TextView) itemView.findViewById(R.id.subtitle);
      selectedOverlay = itemView.findViewById(R.id.selected_overlay);

      this.listener = listener;

      itemView.setOnClickListener(this);
      itemView.setOnLongClickListener(this);
    }

    @Override
    public void onClick(View v) {
      if (listener != null) {
        listener.onItemClicked(getPosition());
      }
    }

    @Override
    public boolean onLongClick(View v) {
      if (listener != null) {
        return listener.onItemLongClicked(getPosition());
      }

      return false;
    }

    public interface ClickListener {
      public void onItemClicked(int position);
      public boolean onItemLongClicked(int position);
    }
  }
}
```
我们用`ActionMode`来区分选择模式和正常模式，当选择模式被激活时显示不同的`ActionBar`。实现`ActionMode.Callback`来激活选择模式。简单起见，`Activity`使用一个内部类来实现其接口，同时实现我们我们新创建的Click接口`Adapter.ViewHolder.ClickListener`。我们需要`Callback`来访问`Adapter `，所以把`Adapter`作为`Activity`的一个属性。
我们的Activity看上去变得复杂了。
```java
public class MainActivity extends ActionBarActivity implements Adapter.ViewHolder.ClickListener {
  @SuppressWarnings("unused")
  private static final String TAG = MainActivity.class.getSimpleName();
  private Adapter adapter;
  private ActionModeCallback actionModeCallback = new ActionModeCallback();
  private ActionMode actionMode;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    adapter = new Adapter(this);

    RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
    recyclerView.setAdapter(adapter);
    recyclerView.setItemAnimator(new DefaultItemAnimator());
    recyclerView.setLayoutManager(new StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL));
  }

  @Override
  public void onItemClicked(int position) {
    if (actionMode != null) {
      toggleSelection(position);
    }
  }

  @Override
  public boolean onItemLongClicked(int position) {
    if (actionMode == null) {
      actionMode = startSupportActionMode(actionModeCallback);
    }
    toggleSelection(position);
    return true;
  }

  /**
   * 触发Item的选中状态
   *
   * 如果选中的Item数量为0, 则关闭选中状态模式
   * 注意：actionMode不能为空
   *
   * @param position 触发选中状态的Item位置
   */
  private void toggleSelection(int position) {
    adapter.toggleSelection(position);
    int count = adapter.getSelectedItemCount();

    if (count == 0) {
      actionMode.finish();
    } else {
      actionMode.setTitle(String.valueOf(count));
      actionMode.invalidate();
    }
  }

  private class ActionModeCallback implements ActionMode.Callback {
    @SuppressWarnings("unused")
    private final String TAG = ActionModeCallback.class.getSimpleName();

    @Override
    public boolean onCreateActionMode(ActionMode mode, Menu menu) {
      mode.getMenuInflater().inflate (R.menu.selected_menu, menu);
      return true;
    }

    @Override
    public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
      return false;
    }

    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
      switch (item.getItemId()) {
        case R.id.menu_remove:
          // TODO: 具体的移除Item操作
          Log.d(TAG, "menu_remove");
          mode.finish();
          return true;

        default:
          return false;
        }
      }

      @Override
      public void onDestroyActionMode(ActionMode mode) {
        adapter.clearSelection();
        actionMode = null;
      }
  }
}

```
下图显示`Item5`被选中时的截图

<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548622/android/recyclerview-selection.png" style="width:35%;">
</div>

### 七、改变数据
想要更新View，，`Adapter`必须来发出数据发生了变化的通知。在`ListView`上，adapter只有一个可以使用的方法`notifyDataSetChanged()`。然而这个方法还不很理想：需要刷新所有的View，因为我们不能够确切知道什么发生了变化。而`RecyclerView.Adapter`提供多种方法：

- notifyItemChanged(int position)
- notifyItemInserted(int position)
- notifyItemRemoved(int position)
- notifyItemMoved(int fromPosition, int toPosition)
- notifyItemRangeChanged(int positionStart, int itemCount)
- notifyItemRangeInserted(int positionStart, int itemCount)
- notifyItemRangeRemoved(int positionStart, int itemCount)
- notifyDataSetChanged()

我们可以知道Item或Items的变化、添加、移除、移动。
我们在`Adapter`中向外部类提供移除Item、Item List的两个`public`方法。如果直接移除一个Item，直接从列表移除即可。但是移除item list 并没有简单。举个例子，如果现在需要移除`[5 ,  8 , 9]` 三个位置的Item，当移除5位置的Item后，我们Item的list长度就会变短，因此list数据中原来的8变成了7。结果我们移除`[5 , 8 , 9]`三个位置的数据实际对应的是[5 , 7 , 7]。再考虑到一种情况如果是移除`[8 ， 9 ， 5]`三个位置的Item呢？
因此我们需要将提供需要移除的Item List进行逆序排序。这样我们就能简单调用`notifyItemRangeRemoved()`方法来移除item list。
```java
public class Adapter extends SelectableAdapter<Adapter.ViewHolder> {
    // …

  public void removeItem(int position) {
    items.remove(position);
    notifyItemRemoved(position);
  }

  public void removeItems(List<Integer> positions) {
    // list逆序排序
    Collections.sort(positions, new Comparator<Integer>() {
      @Override
      public int compare(Integer lhs, Integer rhs) {
        return rhs - lhs;
      }
    });

    // 拆分排列后的list
    while (!positions.isEmpty()) {
      if (positions.size() == 1) {
        removeItem(positions.get(0));
        positions.remove(0);
      } else {
        int count = 1;
        while (positions.size() > count && positions.get(count).equals(positions.get(count - 1) - 1)) {
          ++count;
        }

        if (count == 1) {
          removeItem(positions.get(0));
        } else {
          removeRange(positions.get(count - 1), count);
        }

        for (int i = 0; i < count; ++i) {
          positions.remove(0);
        }
      }
    }
  }

  private void removeRange(int positionStart, int itemCount) {
    for (int i = 0; i < itemCount; ++i) {
      items.remove(positionStart);
    }
    notifyItemRangeRemoved(positionStart, itemCount);
  }

  // …
}
```
最后在`Activity`中调用这两个方法，在`menu`菜单中创建`Remove`的操作。我们只需要在`Remove`菜单中调用`removeItems()`方法就可以了。我们可以在`click`中调用`removeItem()`方法来测试移除单个Item。

```java
public class MainActivity extends ActionBarActivity implements Adapter.ViewHolder.ClickListener {
  // …

  @Override
  public void onItemClicked(int position) {
    if (actionMode != null) {
      toggleSelection(position);
    } else {
      adapter.removeItem(position);
    }
  }

  // …

  private class ActionModeCallback implements ActionMode.Callback {
    // …

    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
      switch (item.getItemId()) {
        case R.id.menu_remove:
          adapter.removeItems(adapter.getSelectedItems());
          mode.finish();
          return true;

        default:
          return false;
      }
    }
    // …
  }
}
```
这样我们就实现了移除Item功能了。与传统`notifyDataSetChanged()`的方法相比，我们还有额外的动画效果


<div  style="text-align:center">
	<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1456548665/android/recyclerview-remove.gif" style="width:35%;">
</div>

### 八、总结
`RecyclerView`使用相对`ListView`和`GridView`有一点复杂，但是提供了 viewholders、数据源、list等之间拆分的简单实现，动画效果提升了用户体验。
关于性能，对比`ListView`，两个都相似的实现了`ViewHolder`模式。然而用`RecyclerView`，你不得不选择`ViewHolder`模式。对于用户来说比没有使用`ViewHolder`有更好的体验，对于开发者来说，可以写出更好的代码。
同时，`RecyclerView`仍然有一些明显的不足（如：StaggeredGridLayoutManager），除此以外可以完美在项目中使用。

示例代码：[GitHub](https://github.com/Kernald/recyclerview-sample)