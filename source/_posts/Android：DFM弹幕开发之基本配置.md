title: Android：DFM弹幕开发之基本配置
date: 2016-09-19 09:34:37
thumbnailImage:  http://res.cloudinary.com/dmfz9aun7/image/upload/b_rgb:fff,c_fit,dpr_0.75,e_grayscale,w_564/v1474249270/android/bilibili_dfm.png
tags: danmaku
categories: danmaku
---
[DanmakuFlameMaster](https://github.com/Bilibili/DanmakuFlameMaster)(以下简称DFM)是哔哩哔哩开源的Android弹幕解析绘制引擎项目。`DFM`使用多种方式(View/SurfaceView/TextureView)实现高效绘制，支持自定义字体，支持多种弹幕参数设置等

# 一、配置

Gradle

```java
repositories {
    jcenter()
}

dependencies {
    compile 'com.github.ctiao:DanmakuFlameMaster:0.4.9'
}
```
在布局中应用

```xml
<master.flame.danmaku.ui.widget.DanmakuView
    android:id="@+id/sv_danmaku"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

# 二、基础用法

## 1、弹幕播放

```java

// Activity onCreate 中初始化弹幕的配置
final DanmakuView svDanmaku = (DanmakuView) findViewById(R.id.sv_danmaku);

HashMap<Integer, Integer> maxLinesPair = new HashMap<>(); // 最大行数
maxLinesPair.put(BaseDanmaku.TYPE_SCROLL_RL, 5);

HashMap<Integer, Boolean> overlappingEnablePair = new HashMap<>(); // 防止弹幕重叠
overlappingEnablePair.put(BaseDanmaku.TYPE_SCROLL_RL, true);
overlappingEnablePair.put(BaseDanmaku.TYPE_FIX_TOP, true);

DanmakuContext mDanmakuContext = DanmakuContext.create();   // 创建弹幕所需的上下文信息
mDanmakuContext.setDanmakuStyle(IDisplayer.DANMAKU_STYLE_STROKEN, 3) // 弹幕样式
        .setDuplicateMergingEnabled(true)    // 合并重复弹幕
        .setScrollSpeedFactor(1.5f)          // 滚动弹幕速度系数
        .setScaleTextSize(0.8f)              // 弹幕文字大小
        .setCacheStuffer(new SpannedCacheStuffer(), mCacheStufferAdapter) // 缓存绘制
        .setMaximumLines(maxLinesPair)       // 弹幕最大行数
        .preventOverlapping(overlappingEnablePair); // 防止弹幕重叠

BaseDanmakuParser mParser = createParser(this.getResources().openRawResource(R.raw.comments)); // 创建弹幕解析器
// 设置弹幕播放的状态回调
svDanmaku.setCallback(new DrawHandler.Callback() {
    @Override
    public void prepared() {
        // 弹幕准备完成
        Log.d("DFM" , "prepared");
        svDanmaku.start();
    }

    @Override
    public void updateTimer(DanmakuTimer timer) {
        // 弹幕播放时间
        Log.d("DFM" , "updateTimer : timer = " + timer.currMillisecond);
    }

    @Override
    public void danmakuShown(BaseDanmaku danmaku) {
        // 播放新的一条弹幕
        Log.d("DFM" , "danmakuShown : danmaku = " + danmaku.text);
    }

    @Override
    public void drawingFinished() {
        // 弹幕播放结束
        Log.d("DFM" , "drawingFinished");
    }});
    
    // 设置弹幕点击监听
    mMainBinding.svDanmaku.setOnDanmakuClickListener(new IDanmakuView.OnDanmakuClickListener() {
        @Override
        public boolean onViewClick(IDanmakuView view) {
            // 点击的弹幕
            return false;
        }

        @Override
        public boolean onDanmakuClick(IDanmakus danmakus) {
            // 弹幕
            Log.d("DFM", "onDanmakuClick danmakus size:" + danmakus.size());
            return false;
        }
    });

    // 播放准备
    svDanmaku.prepare(mParser, mDanmakuContext);
    // 显示播放FPS,一般用作调试时使用
    svDanmaku.showFPS(false);
    // 开启绘制缓存
    svDanmaku.enableDanmakuDrawingCache(true);

```
创建弹幕解析器
```java
    /**
     * 创建弹幕解析器
     * @param stream stream
     * @return BaseDanmakuParser
     */
    private BaseDanmakuParser createParser(InputStream stream) {

        if (stream == null) {
            return new BaseDanmakuParser() {
                @Override
                protected Danmakus parse() {
                    return new Danmakus();
                }
            };
        }
        // TAG_BILI XML数据格式的弹幕。TAG_ACFUN JSON数据格式的弹幕
        ILoader loader = DanmakuLoaderFactory.create(DanmakuLoaderFactory.TAG_BILI);

        try {
            loader.load(stream);
        } catch (IllegalDataException e) {
            e.printStackTrace();
        }
        BaseDanmakuParser parser = new BiliDanmukuParser();
        IDataSource<?> dataSource = loader.getDataSource();
        parser.load(dataSource);
        return parser;
    }
```
![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1474278152/android/dfm_play.gif)
## 2、发送弹幕
常见的弹幕内容一般由以下部分组成
```java
1 : 时间(弹幕出现时间) long类型的时间戳
2 : 类型(1从左至右滚动弹幕 ； 6从右至左滚动弹幕； 5顶端固定弹幕；4底端固定弹幕；7高级弹幕；8脚本弹幕)
3 : 字号
4 : 颜色
5 : 弹幕池id
6 : 用户hash
7 : 弹幕id
```
发送弹幕处理
```java
BaseDanmaku baseDanmaku = mDanmakuContext.mDanmakuFactory.createDanmaku(BaseDanmaku.TYPE_FIX_BOTTOM); // 通过DanmakuFactory来创建弹幕

// 弹幕内容信息
baseDanmaku.text = "简单的一条弹幕" + System.nanoTime(); // 文本内容
baseDanmaku.textSize = 25f * (mParser.getDisplayer().getDensity() - 0.6f); // 字体大小
baseDanmaku.textColor = Color.RED;         // 字体颜色
baseDanmaku.textShadowColor = Color.WHITE; // 描边颜色
baseDanmaku.borderColor = Color.RED;       // 弹幕边框颜色
baseDanmaku.padding = 5;                   // 内边距
baseDanmaku.priority = 0;                  // 弹幕优先级,0为低优先级
baseDanmaku.isLive = false;                // 是否是直播弹幕
baseDanmaku.underlineColor = 0;            // 0表示无下划线
// baseDanmaku.lines = new String[]{"两行弹幕第1行弹幕" , "两行弹幕第2行弹幕"};

// 发送时的时间由弹幕播放时间来确定
baseDanmaku.setTime(mMainBinding.svDanmaku.getCurrentTime());// 弹幕出现的时间

// 发送弹幕，根据业务内容是否执行弹幕上传操作 TODO
mMainBinding.svDanmaku.addDanmaku(baseDanmaku);
```

![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1474278810/android/dfm_add.gif)

## 3. 弹幕状态操作
（1）. 隐藏
```java
mMainBinding.svDanmaku.hide();
```
（2）. 显示
```java
mMainBinding.svDanmaku.show();
```
（3）. 暂停
```java
mMainBinding.svDanmaku.pause();
```
（4）. 恢复继续
```java
mMainBinding.svDanmaku.onResume();
```
（2）. 跳转
```java
mMainBinding.svDanmaku.seekTo(0);   // 跳转到起始位置
```
## 4. 弹幕周期管理及资源释放
弹幕需要与Activity的生命周期配合来处理。防止未及时释放资源，造成Activity的内存问题。
```java

@Override
protected void onPause() {
    super.onPause();
    if (mDanmakuView != null && mDanmakuView.isPrepared()) {
        mDanmakuView.pause();
    }
}

@Override
protected void onResume() {
    super.onResume();
    if (mDanmakuView != null && mDanmakuView.isPrepared() && mDanmakuView.isPaused()) {
        mDanmakuView.resume();
    }
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (mDanmakuView != null) {
        // 及时释放弹幕资源
        mDanmakuView.release();
        mDanmakuView = null;
    }
}

@Override
public void onBackPressed() {
    super.onBackPressed();
    if (mDanmakuView != null) {
        // 及时释放弹幕资源
        mDanmakuView.release();
        mDanmakuView = null;
    }
}

```
# 二、高级弹幕

## 1. 背景绘制

[DFM](https://github.com/Bilibili/DanmakuFlameMaster)自定义SpannedCacheStuffer，drawBackground方法中实现背景绘制操作

```java
// 绘制有背景的弹幕
private class BackgroundCacheStuffer extends SpannedCacheStuffer{
    final Paint mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);

    @Override
    public void measure(BaseDanmaku danmaku, TextPaint paint, boolean fromWorkerThread) {
        danmaku.padding = 10;
        super.measure(danmaku, paint, fromWorkerThread);
    }

    @Override
    public void drawBackground(BaseDanmaku danmaku, Canvas canvas, float left, float top) {
        mPaint.setColor(0xEE90A4AE);   // 背景颜色
        RectF rectF = new RectF(left + 5, top + 5, left + danmaku.paintWidth - 5, top + danmaku.paintHeight - 5); // 圆角矩形
        canvas.drawRoundRect(rectF, 50 , 50 , mPaint);
    }

    @Override
    public void drawStroke(BaseDanmaku danmaku, String lineText, Canvas canvas, float left, float top, Paint paint) {
        // 绘制背景时建议禁止绘制描边
    }
}
```
配置DanmakuContext
```java
private DanmakuContext mDanmakuContext;
...
mDanmakuContext.setDanmakuStyle(IDisplayer.DANMAKU_STYLE_STROKEN, 3) // 弹幕样式
        .setDuplicateMergingEnabled(true)    // 合并重复弹幕
        .setScrollSpeedFactor(1.5f)          // 滚动弹幕速度系数
        .setScaleTextSize(0.8f)              // 弹幕文字大小
        .setCacheStuffer(new BackgroundCacheStuffer(), mCacheStufferAdapter) // 缓存绘制
        ...
```
![img](http://res.cloudinary.com/dmfz9aun7/image/upload/c_scale,w_521/v1474442790/android/danmaku_bg.png)

## 2. 自定义解析器

[DFM](https://github.com/Bilibili/DanmakuFlameMaster)内置两种解析器[BiliDanmukuParser](https://github.com/Bilibili/DanmakuFlameMaster/blob/d4abc40bcaa8cc6786c1dff4ca1694dd626955b5/Sample/src/main/java/com/sample/BiliDanmukuParser.java)。`BiliDanmukuParser`使用XML数据格式的弹幕，是BiliBili标准的弹幕格式。`AcFunDanmakuParser`使用JSON数据格式的弹幕，适用于解析Acfun标准的弹幕。除了使用这两种弹幕格式以外，还可以根据具体弹幕内容来自定义弹幕解析器。如有以下内容Json格式的弹幕，那么如何来处理这种数据呢，此时就需要自定义实现弹幕数据的解析。
```java
[{
    "content":   "弹幕内容" ,      // 弹幕显示的内容
    "time" :     23.826000213623, // 弹幕出现的时间
    "type":       6      ,        // 弹幕类型（参考danmaku中弹幕类型）
    "fontsize":   12     ,        // 弹幕显示的字体大小
    "fontcolor":  16777215,       // 弹幕字体颜色
    "userid":     6,              // 用户hash
    "username":   "alexwan",      // 用户名称
    "avatar":     "http://res.cloudinary.com/dmfz9aun7/image/upload/v1467081842/android/3397490934332211.jpg",
    "danmakuid":  90              // 弹幕id
}, 
...
]
```
（1）定义弹幕JavaBean
```java
public class DanmakuData {
    private String content;
    private double time;
    private int type;
    private float fontSize;
    private int fontColor;
    private int userId;
    private String userName;
    private String avatar;
    private int danmakuId;
    ...
}
```
（2）实现[IDataSource](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/parser/IDataSource.java)，定义弹幕数据源（参考[JSONSource](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/parser/android/JSONSource.java)），加载指定数据源的弹幕数据。在[ILoader](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/loader/ILoader.java)子类中初始化。
{% codeblock lang:java DanmakuSource.java %}

public class DanmakuSource implements IDataSource<String> 
...
private String mJson;
private InputStream mInput;
public DanmakuSource(String json) throws JSONException{...}                      // 指定json字符串
public DanmakuSource(InputStream in) throws JSONException{...}                   // 指定json数据流
public DanmakuSource(URL url) throws JSONException, IOException{...}             // 指定URL源json
public DanmakuSource(File file) throws FileNotFoundException, JSONException{...} // 指定file源json
public DanmakuSource(Uri uri) throws IOException, JSONException{...}             // 指定Uri源json
private void init(InputStream in) throws JSONException 
...
private void init(String json) throws JSONException {
    if (!TextUtils.isEmpty(json)) {
        // 初始化json
        mJson = json;
    }
}
...
@Override
public String data() {
    return mJson;
}
...
{% endcodeblock %}


（3）实现[ILoader](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/loader/ILoader.java)，执行`IDataSource `的初始化管理操作。XML格式弹幕直接使用[BiliDanmakuLoader](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/loader/android/BiliDanmakuLoader.java)加载，JSON格式弹幕可以使用[AcFunDanmakuLoader](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/loader/android/AcFunDanmakuLoader.java)用`JSONSource`来加载弹幕数据。也可以自定义实现弹幕加载方式。
{% codeblock lang:java SimpleDanmakuLoader.java %}
public class SimpleDanmakuLoader implements ILoader {

    private SimpleDanmakuLoader() {
    }
	
    private static volatile SimpleDanmakuLoader instance;
    private IDataSource<String> mDataSource;
	
    //  这里使用单例保证加载器只会被实例化一次
    public static ILoader instance() {
        if (instance == null) {
            synchronized (SimpleDanmakuLoader.class) {
                if (instance == null) {
                    instance = new SimpleDanmakuLoader();
                }
            }
        }
        return instance;
    }

    @Override
    public IDataSource<String> getDataSource() {
        return mDataSource;
    }

    /*
    * 加载Uri类型弹幕数据
    */
    @Override
    public void load(String uri) throws IllegalDataException {
        try {
             mDataSource = new DanmakuSource(UriUtil.parseUriOrNull(uri));
        } catch (Exception e) {
            throw new IllegalDataException(e);
        }
    }

    /**
    * 加载流数据弹幕
    */
    @Override
    public void load(InputStream in) throws IllegalDataException {
        try {
            mDataSource = new DanmakuSource(in);
        } catch (JSONException e) {
            throw new IllegalDataException(e);
        }
    }
}
{% endcodeblock %}
`SimpleDanmakuLoader`创建`DanmakuSource`来获取弹幕内容。在初始化弹幕数据解析器时，调用`load`方法获取弹幕数据。

（4）定义弹幕解析类，继承[BaseDanmakuParser](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/DanmakuFlameMaster/src/main/java/master/flame/danmaku/danmaku/parser/BaseDanmakuParser.java)创建自定义的弹幕解析类。
{% codeblock lang:java SimpleDanmakuParser.java%}
public class SimpleDanmakuParser extends BaseDanmakuParser{
    private static final String TAG = SimpleDanmakuParser.class.getSimpleName();
    @Override
    protected IDanmakus parse() {
        if(mDataSource != null && mDataSource instanceof DanmakuSource){
            DanmakuSource source = (DanmakuSource) mDataSource;
            return doParse(source.data());
        }
        return new Danmakus();
    }

    private Danmakus doParse(String json){
        Danmakus danmakus = new Danmakus();
        if(TextUtils.isEmpty(json)){
            return danmakus;
        }
        // 使用Gson解析弹幕json数据
        Type type = new TypeToken<List<DanmakuData>>(){}.getType();
        List<DanmakuData> list = new Gson().fromJson(json , type);
        int size = list.size();
        for (int i = 0 ; i < size; i ++){
            danmakus = parse(list.get(i) , danmakus , i);
        }
        return danmakus;
    }

    /**
     * 解析弹幕数据,组装BaseDanmaku
     * @param data DanmakuData
     * @param danmakus Danmakus
     * @param index index
     * @return Danmakus
     */
    private Danmakus parse(DanmakuData data, Danmakus danmakus , int index) {
        if(danmakus == null){
            danmakus =  new Danmakus();
        }
        BaseDanmaku item = mContext.mDanmakuFactory.createDanmaku(data.getType() , mContext);
        if(item != null){
            item.setTime((long) (data.getTime() * 1000));
            item.textSize = data.getFontSize();
            item.textColor = data.getFontColor() | 0xFF000000 ;
            DanmakuUtils.fillText(item , data.getContent());
            item.index = index;
            item.setTimer(mTimer);
            danmakus.addItem(item);
            Log.i(TAG , "parse : time = " + data.getTime() * 1000 + " ; textSize = " + data.getFontSize()
                    + " ; textColor = " + item.textColor + " ; content = " + item.text);
        }
        return danmakus;
    }
{% endcodeblock %}

（5）配置弹幕解析器

{% codeblock lang:java MainActivity.java %}
...
mParser = customParser();
mDanmakuView.prepare(mParser, mDanmakuContext);
...
private BaseDanmakuParser customParser(){
    ILoader loader = SimpleDanmakuLoader.instance();
    try {
        loader.load("http://...");
        Log.i(TAG , "customParser : json = " + new Gson().toJson(list) );
    } catch (IllegalDataException e) {
        Log.e(TAG , "customParser : json = " + new Gson().toJson(list) + " ; error = " , e);
    }
    BaseDanmakuParser parser = new SimpleDanmakuParser();
    IDataSource<?> dataSource = loader.getDataSource();
    parser.load(dataSource);
    return parser;
}
{% endcodeblock %}

![img](http://res.cloudinary.com/dmfz9aun7/image/upload/c_scale,w_556/v1474625129/android/danmaku_parser.png)
## 3. 图文混排弹幕

（1）单排图文
```java
// TODO


```
（2）双排图文

```java
// TODO
```

# 参考
1、[Android开源弹幕引擎·烈焰弹幕使 ～ ](https://github.com/Bilibili/DanmakuFlameMaster/blob/master/Sample/src/main/java/com/sample/MainActivity.java)
2、[记一次弹幕的开发](http://wangpeiyuan.cn/2016/02/24/%E8%AE%B0%E4%B8%80%E6%AC%A1%E5%BC%B9%E5%B9%95%E7%9A%84%E5%BC%80%E5%8F%91/)
3、[图文弹幕的实现](http://blog.csdn.net/zxq614/article/details/52622792)
4、[Simple drawee spannable text view based on Fresco](https://github.com/Bilibili/drawee-text-view)




