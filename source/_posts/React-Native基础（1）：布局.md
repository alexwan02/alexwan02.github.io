---
title: React-Native基础（1）：布局
date: 2017-05-16 21:38:50
tags: react-native
categories: react-native
---
## Flex布局
Flexbox布局旨在提供一个更佳有效的布局方式，更好的控制项目的对齐和自由分配容器空间，即使它们的大小是未知的或动态的。因此得其名"flex"。
### 父容器属性
- flexDirection?: enum('row', 'column','row-reverse','column-reverse')
- flexWrap?: enum('wrap', 'nowrap')
- justifyContent?: enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around')
- alignItems?: enum('flex-start', 'flex-end', 'center', 'stretch')

### 主轴与侧轴
![](http://img-1252300500.cosgz.myqcloud.com/react-native/layout/flexbox_main-cross-axis.png)
Flex模块方向分为主轴方向与侧轴方式，之所以不叫水平方向或垂直方向，是因为主轴和侧轴要依据flexDirection来决定。
1. 当为`row`或者`row-reverse`时，水平方向是主轴，垂直方向是侧轴；
2. 当为`column`或`column-reverse`时，水平方向是侧轴，垂直方向是主轴；

- flexDirection?: enum('row', 'column','row-reverse','column-reverse')
控制子元素水平和垂直方向。水平方向默认为`row`，垂直方向默认为`column`。`flexDirection`决定了当前容器的主轴和侧轴。

row-reverse | column-reverse
---|---
![row-reverse](http://img-1252300500.cosgz.myqcloud.com/react-native/layout/flexDirection_row_reverse.png) | ![column-reverse](http://img-1252300500.cosgz.myqcloud.com/react-native/layout/flexDirection_column_reverse.png)

- flexWrap?: enum('wrap', 'nowrap')
控制子元素超过容器尺寸时的显示方式

wrap | nowrap
--- | ---
![wrap](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/flexWrap_wrap.png) | ![nowrap](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/flexWrap_nowrap.png)

- justifyContent?: enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around') 
定义在主轴方向上排列子元素的方式。

flex-start | flex-end | center 
---|--- |---
![flex-start](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/justifyContent_flex-start.png) | ![flex-end](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/justifyContent_flex-end.png) | ![center](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/justifyContent_center.png) 
space-between | space-around | 
![space-between](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/justifyContent_space-between.png) | ![space-around](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/justifyContent_flex-around.png)

- alignItems?: enum('flex-start', 'flex-end', 'center', 'stretch', 'baseline')
定义子元素在容器中侧轴方向上的排列方式。与`justifyContent`相似，但是与`justifyContent`轴不相同。

flex-start |  flex-end | center | base-line | stretch
---|--- | --- |--- |---
![img](http://img-1252300500.cosgz.myqcloud.com/react-native/alginContent_flex-start.png) | ![img](http://img-1252300500.cosgz.myqcloud.com/react-native/alginContent_flex-end.png) | ![img](http://img-1252300500.cosgz.myqcloud.com/react-native/alginContent_center.png) | ![img](http://img-1252300500.cosgz.myqcloud.com/react-native/alginContent_baseline.png) |![img](http://img-1252300500.cosgz.myqcloud.com/react-native/alginContent_stretch.png)

- `alignContent`?: enum('flex-start', 'flex-end', 'center', 'stretch', 'space-between', 'space-around')
定义每排在侧轴上的排列方式，当只有一排时属性无作用。

flext-start | flex-end | center
:---:|:---:|:---: |:---:
![flext-start](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_flex-start.png) | ![flex-end](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_flex-end.png) | ![center](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_center.png)
stretch | space-between | space-around
![stretch](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_stretch.png) | ![space-between](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_space-between.png) | ![space-around](http://img-1252300500.cosgz.myqcloud.com/react-native/alignContent_space-around.png)

### 子元素属性
- alignSelf?: enum('auto', 'flex-start', 'flex-end', 'center', 'stretch', 'baseline')
定义Flex容器内子元素在侧轴上的排列方式，覆盖父容器的`alignItems`属性。
    - auto 表示继承了它的父容器的`alignItems`属性，无父容器默认为`stretch`拉伸。
    - 其他属性参照`alignItems`属性。

- flex?: number 
定义组件伸缩能力大小，默认为0。在React-native中，flex与css中效果不同，值为数值。
    - 值为正数时，由值大小来定义尺寸的大小。如flex为2的元素要比felx为1的元素大两倍。
    - 值为0时，组件大小由`width`和`height`决定
    - 值为-1时，组件大小通常由`width`和`height`决定，然而在没有足够空间时，组件会缩小到`minWidth`和`minHeight`.
    - flexGrow, flexShrink和flexBasis效果与CSS中效果一致。
- flexBasis?: number, string 

- flexGrow?: number
定义Flex容器Item的扩展因数，指定Item能够占据的空间值，默认`auto`

![flexGrow](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/flex-grow.png)
- flexShrink?: number
定义Flex容器Item的缩小因数，当Items的默认宽度超过容器宽度时，按照flexShrink值缩放items填充容器
![flexShrink](http://imgbucket-1252300500.picsh.myqcloud.com/react/style/flex-shrink.png)

- aspectRatio?: number 
宽高比非CSS标准属性，只适应于react-native，用于控制未定义元素的尺寸大小。

## 尺寸
- height?: number, string 
- width?: number, string
- maxHeight?: number, string 
- maxWidth?: number, string
- minHeight?: number, string
- minWidth?: number, string

## 边框
- borderBottomWidth?: number 
- borderLeftWidth?: number
- borderRightWidth?: number
- borderTopWidth?: number
- borderWidth?: number

## 边距

- top?: number, string
- bottom?: number, string 
- left?: number, string
- right?: number, string 

### 外边距
- margin?: number, string 
- marginTop?: number, string 
- marginBottom?: number, string 
- marginVertical?: number, string 与同时设置`marginBottom`与`marginTop`一样
- marginLeft?: number, string 
- marginRight?: number, string
- marginHorizontal?: number, string 与同时设置`marginLeft`与`marginRight`一样

### 内边距
- padding?: number, string
- paddingTop?: number, string 
- paddingBottom?: number, string
- paddingVertical?: number, string 与同时设置`paddingTop`与`paddingBottom`一致
- paddingLeft?: number, string 
- paddingRight?: number, string
- paddingHorizontal?: number, string 与同时设置`paddingLeft`与`paddingRight`一致

## 其他

- display?: string
设置组件的显示类型，效果类似CSS的display，但只支持`flex`和`none`。默认为`flex`
- overflow?: enum('visible', 'hidden', 'scroll')
定义超出容器时，子元素的绘制和显示方式。
    - visible 裁剪溢出内容
    - hidden 隐藏超过容器的部分
    - scroll 滚动显示溢出内容
- position?: enum('absolute', 'relative')
与CSS类似，但是默认值为`relative`，所以`absolute`只针对相对于父容器有效。
- zIndex?: number 
指定显示在组件层级关系，通常情况来说无须使用zIndex。组件按照其在DOM树中的顺序决定层级关系。
- `[ios]`direction?: enum('inherit', 'ltr', 'rtl')




