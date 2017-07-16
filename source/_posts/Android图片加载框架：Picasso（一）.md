---
title: Android图片加载框架：Picasso（一）
date: 2016-01-28 11:46:43
tags:
---
[Picasso](http://square.github.io/picasso/)
[Picasso JavaDoc](http://square.github.io/picasso/2.x/picasso/)
[Picasso stackOverflow](http://stackoverflow.com/questions/tagged/picasso?sort=active)
### 一、介绍
在Android开发中的视觉显示上，图片占有很大的比重。Picasso可以只使用一行代码来解决图片加载的问题。
```java
Picasso.with(context).load("http://i.imgur.com/DvpvklR.png").into(imageView);
```
Picasso 可以极大减少Android开发过程中的加载图片的问题：
- 解决`ImageView `回收和`Adapter`下载取消的问题。
- 使用最小的内存来处理图片转换问题
- 自动的磁盘和内存缓存

### 二、特性
1、Adapter 图片加载
adapter自动检测是否重用图片，是否取消之前图片的加载。
```java
@Override 
public void getView(int position, View convertView, ViewGroup parent) {
    SquaredImageView view = (SquaredImageView) convertView;
    if (view == null) {
        view = new SquaredImageView(context);
    }
    String url = getItem(position);
    Picasso.with(context).load(url).into(view);
}
```

2.图片转换

(1) 转换图片来满足Layout的合适大小以降低内存。
```java
Picasso.with(context)
  .load(url)
  .resize(50, 50)
  .centerCrop()
  .into(imageView)
```
(2) 指定自定义的变换适应更多的效果
```java
public class CropSquareTransformation implements Transformation {
    @Override
    public Bitmap transform(Bitmap source) {
        int size = Math.min(source.getWidth(), source.getHeight());
        int x = (source.getWidth() - size) / 2;
        int y = (source.getHeight() - size) / 2;
        Bitmap result = Bitmap.createBitmap(source, x, y, size, size);
        if (result != source) {
            source.recycle();
        }
        return result;
    }

    @Override 
    public String key() { 
        return "square()"; 
    }
}
返回值指定的对象给`transform`方法
```
3.Place Holders
Picasso支持设置默认和加载错误图片
```java
Picasso.with(context)
    .load(url)
    .placeholder(R.drawable.user_placeholder)
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```
在显示加载错误的图片之前会尝试请求三次指定图片的地址。

4.资源加载
Picasso支持Resources，assets，files，content providers 等图片资源
```java
Picasso.with(context).load(R.drawable.landing_screen).into(imageView1);
Picasso.with(context).load("file:///android_asset/DvpvklR.png").into(imageView2);
Picasso.with(context).load(new File(...)).into(imageView3);
```
5.Debug指示器
作为开发者，可以设置是否使用颜色指示器来显示加载图片的来源：网络、磁盘、内存。
```java
// Picasso instance
setIndicatorsEnabled(true)
```
6.配置

- maven

```xml
<dependency>
  <groupId>com.squareup.picasso</groupId>
  <artifactId>picasso</artifactId>
  <version>2.5.2</version>
</dependency>
```
- gradle

```java
compile 'com.squareup.picasso:picasso:2.5.2'
```