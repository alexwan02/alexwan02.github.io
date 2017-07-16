title: Android：SurfaceView基础与原理
date: 2016-09-06 10:47:41
tags:
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1467529028/android/8ff23095a2a4e04af26ca63642bfdea3_b.png
---
普通的View以及派生类都是共享同一个surface的，所有的绘制都必须在UI线程中执行。而SurfaceVAiew比较特殊，并不与其他普通View共享Surface，而是在内部持有了一个独立的surface。SurfaceView负责管理这个Surface的格式，尺寸以及显示位置。由于UI线程还要同事处理其他交互逻辑，因此对View的更新速度和帧率无法保证，而SurfaceView由于持有一个独立Surface，可以在独立线程中进行绘制，提供更高的帧率。自定义相机的预览图像由于
