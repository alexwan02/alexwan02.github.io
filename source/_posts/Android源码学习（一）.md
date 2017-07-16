---
title: Android源码学习（一）：Mac OSX开发环境搭建
date: 2016-01-12 16:41:51
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1457059153/android/AOSP1.jpg
categories: Android源码学习系列
tags: android

---

[官网教程](https://source.android.com/source/initializing.html#setting-up-a-mac-os-x-build-environment)

[Mac OS X 10.10.3下android-5.1.1_r9 源码下载与编译](http://iluhcm.com/2015/08/18/compile-newest-android-source-code-on-macosx/)

[Build Android 5.0 Lollipop on OSX 10.10 Yosemite](https://medium.com/@raminmahmoodi/build-android-5-0-lollipop-on-osx-10-10-yosemite-441bd00ee77a#.vxeycv4xr)

[Mac 10.10 编译android 4.4.4 for nexus](http://www.liball.me/mac-10-10-build-android-4-4-4-for-nexus/)

[Build Android 5.0 Source](http://llzz.me/2015/06/17/Build-Android-5-0-Source/)
#### 一、Mac OSX 开发环境搭建
本文的Mac OSX的版本为 `OSX 10.11 `

默认情况下，MacOs系统是大小写保留，但大小写不敏感的文件系统。这种文件系统不被git支持，会引起git一些命令行行为不正常，如 `git status`。因此建议将ASOP源码文件放在一个大小写敏感的工作环境中。可以使用磁盘镜像来建立一个这样的文件系统。
#### 1、创建磁盘文件
在MacOs系统中使用磁盘镜像创建一个大小写敏感的文件系统很简单。打开Mac自带的磁盘工具（Disk Utility），选择`New Image`，分配至少26G大小的空间；稍微大一点的空间可能会更好如`50G`。保证选择格式为`OSX 扩展（区分大小写，日志式）`的卷。

<div>
	<img src= "http://res.cloudinary.com/dmfz9aun7/image/upload/v1457867135/android/android_disk_img.jpg" style="height:350px">
</div>

也可以通过shell命令行来创建磁盘镜像

```java
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 50g ~/android.dmg
```
上面的命令行会创建一个`.dmg`（或`.dmg.sparseimage`）格式文件。以后可以用这个镜像来作为Android 开发环境的磁盘。
之后，如果需要更大的磁盘空间，可以用下面的命令行来改变磁盘镜像的大小
```java
# hdiutil resize -size <new-size-you-want>g ~/android.dmg.sparseimage
```
对于名称为`android.dmg`的磁盘镜像，可以在`~/.bash_profile`文件中加入下面的辅助`function`来启动你的磁盘镜像

- 当执行`mountAndroid`时挂载镜像
```java
# mount the android file image
function mountAndroid { hdiutil attach ~/android.dmg -mountpoint /Volumes/android; }
```
- 执行`umountAndroid`卸载镜像
```java
# unmount the android file image
function umountAndroid() { hdiutil detach /Volumes/android; }
```
镜像文件的地址要视你的本地具体的镜像的位置（本地的镜像地址在home的根目录中）。
磁盘镜像创建完毕

#### 2、安装配置JDK
在安装和配置JDK的时候要根据你的Mac 系统的版本来决定你要选择的JDK版本和要编译的Android Source的分支（branch）。

- `master`分支的[AOSP](https://android.googlesource.com/)需要Java 8.x 对应的[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html#jdk-8u45-oth-JPR)
- `5.0.x`分支的AOSP需要Java 7对应的版本[JDK 1.7](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u71-oth-JPR)
- `Gingerbread `到`KitKat `版本之间的Android版本需要下载安装Java 6 对应的 [Java JDK](http://support.apple.com/kb/dl1572)

##### 1) Master分支
为了在Mac上编译最新的源码，需要运行`Mac OS X v10.10 (Yosemite)`或`更新`版本的Mac，和对应的`Xcode 4.5.2`及以上版本的Command Line Tools。
##### 2) 6.0.x分支
编译 6.0.x的AOSP 源码，需要系统版本为`Mac OS X v10.10 (Yosemite)`的Mac，和对应版本为`Xcode 4.5.2 `与`Command Line Tools`
##### 3) 5.0.x分支
编译 5.0.x的AOSP 源码，需要系统版本为`Mac OS X v10.8 (Mountain Lion)`的Mac，和对应版本为`Xcode 4.5.2 `与`Command Line Tools`
##### 4) 4.4.x分支
编译 4.4.x的AOSP 源码，需要系统版本为`Mac OS X v10.6 (Snow Leopard) `或`Mac OS X v10.7 (Lion)`的Mac，和对应版本为`Xcode 4.2`与`Command Line Tools`。
##### 5) 4.0.x以及更早的分支
对于4.0.x以及更早版本的分支，需要系统版本为`Mac OS X v10.5 (Leopard)  `或`Mac OS X v10.6 (Snow Leopard)`的Mac，同时要有`Mac OS X v10.5 SDK`的支持
#### 3、安装需求工具
- 安装Xcode

从[Apple Developer](https://developer.apple.com/resources/cn/)网站来安装Xcode。建议`3.1.4`版本及以上的Xcode。版本`4.x`可能会引起一些问题。如果你未注册苹果开发者，则需要创建一个Apple ID来下载Xcode。

- 安装[MacPorts](http://www.macports.org/install.php)（安装时间可能会比较长，耐心等待即可）

`注意：` 保证在`/usr/bin`上级路径中已有`/opt/local/bin`路径目录。如果没有，需要在` ~/.bash_profile`文件中添加一下命令
```java
export PATH=/opt/local/bin:$PATH
```
`注意：`如果home根目录中不存在.bash_profile 文件，需要手动创建一个。

- 从MacPorts上获取`Git`、`GPG`等包
```java
$ POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg
```
如果Mac系统版本为OS X v10.4，还需要安装`bison `
```java
$ POSIXLY_CORRECT=1 sudo port install bison
```
对于版本`ICS`（ Ice Cream Sandwich (4.0.x)）之前的Android，使用 gmake 3.82版本会有阻止android源码编译的bug，安装时请安装最新的gmake或者通过以下几步安装`3.81`版本。

1. 编辑`/opt/local/etc/macports/sources.conf`，添加一行
```java
file:///Users/Shared/dports
```
2. 在远程同步之前，创建目录
```java
$ mkdir /Users/Shared/dports
```
3. 在新的dports目录中运行
```java
$ portindex /Users/Shared/dports
```
4. 最后安装旧版本的gmake
```java
$ sudo port install gmake @3.81
```
#### 4、解除文件限制
默认情况下，Mac系统限制了同时打开文件数，会限制编译进程的执行；在 ~/.bash_profile文件中添加以下命令来取消限制：
```java
# set the number of open files to be 1024
ulimit -S -n 1024
```
#### 5、自定义配置（可选）
使用`ccache `编译工具来加速`rebuilds `。如果你经常用`make clean `命令或切换不同开发版本，`ccache`很有用。
- 在.bashrc（或类似）文件中加入下面命令
```java
export USE_CCACHE=1
```
默认情况`cache `会被存储在~/.ccache中。如果你的home目录在NFS或其他非本地文件系统上，你也可以在.bashrc文件中指定cache目录
```java
export CCACHE_DIR=<path-to-your-cache-directory>
```
建议缓存的大小为50-100GB，下载源码后需要先执行下面的命令
```java
prebuilts/misc/linux-x86/ccache/ccache -M 50G
```
如果是Mac系统，应该用darwin-x86来替换linux-x86
```java
prebuilts/misc/darwin-x86/ccache/ccache -M 50G
```
当编译` Ice Cream Sandwich (4.0.x)`及之前版本时，`ccache `则在不同的位置：
```java
prebuilt/linux-x86/ccache/ccache -M 50G
```
设置存储在CCACHE_DIR中，并且为持久化状态。
#### 二、下载源码
配置好开发环境后，就可以进行源码下载了。因为国内网络环境，国内开发者建议通过[清华AOSP](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)来下载AOSP。用清华AOSP时建议用`repo git`来下载源码。
##### 1.安装Repo
Repo是一个辅助Git管理Android版本及分支的工具.在安装repo前,需要新建一个文件夹~/bin(名字可随意定)并把这个文件夹放到PATH环境变量里,然后我们就可以把repo下载到这个文件夹里.
- 保证你在home目录中有`bin/`的目录并且包含在你的路径中
```java
$ mkdir ~/bin
$ PATH=~/bin:$PATH
```
- 下载Repo工具并保证可以执行
```java
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
- 建立工作目录（目录可以是任意指定的 ，Mac可以在挂载`Disk Image`中创建目录）
```java
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```
- 配置Git的name 和 email。为了使用Gerrit Code-review工具，需要一个注册[Google账号](https://www.google.com/accounts)的email，保证email是激活状态能够收取到消息。(若为清华AOSP，Email随意)
```java
$ git config --global user.name "Your Name"
$ git config --global user.email "you@example.com"
```
- 运行`repo init`来初始化代码仓库。必须为manifest指定一个URL，指定要下载的远程代码分支
```java
$ repo init -u https://android.googlesource.com/platform/manifest
```
上面初始的是`master`分支的代码，根据你的Mac系统版本来选择android源码版本，可以为之后编译工具减少不少的麻烦。用`-b`命令来选择分支。因为本文Mac系统版本为 Mac OSX 10.11，所以用的master分支的源码来进行源码学习。具体分支查看[Source Code Tags and Builds](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)。
```java
$ repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```
国内开发者可以参考文章[Android 镜像使用帮助](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp)

初始化成功后，会有`Repo is initialized in your working directory`的成功消息了。对应源码根目录会含有对应的`.repo`目录。

- 下载源码：在上面指定`WORKING_DIRECTORY`根目录中用repo sync命令拉取android 源码，并下载到本地，
```java
$ repo sync
```
下载源码的时间特别漫长,中途可能会发生断开连接的现象。不过下载支持断点续传，停止时再用repo sync命令即可。
更多关于[Repo command](https://source.android.com/source/developing.html)

#### 三、编译源码
源码下载完成后就可以进行编译工作了。源码下载成功后，会自动解压缩到`WORKING_DIRECTORY`目录中，如果你下载的源码不是在挂载的android磁盘镜像中，需要将`WORKING_DIRECTORY`中的源码拷贝到`/Volumes/android/`中，进行编译（command+c - command + v 即可）。

- 进入源码根目录
```java
cd /Volumes/android
```
- 执行envsetup.sh脚本初始化环境
	```java
	$ source build/envsetup.sh
	```
	或
	```java
	$ . build/envsetup.sh
	```
得到类似这面这样的提示信息
<div>
<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1457874157/android/envsetup_sh.jpg" style="width:600px">
</div>

- 选择编译的目标
用`lunch `选择要编译的目标，如：
```java
$ lunch aosp_arm-eng
```
表示编译后的版本为模拟器的版本。成功后得到类似下面这样的信息
<div>
<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1457874467/android/lunch_build.jpg" style="width:600px">
</div>

更多查看[Running Builds](https://source.android.com/source/running.html)

- 开始编译
```java
$ make -jN
```
其中N代表同时进行的任务数.官方建议任务数设置为线程数的1~2倍,比如我的机器是单CPU,四核,8线程,则最快的构建任务数是8~16。根据不同机器的性能，编译时长会有不同的差异。本文的Mac配置`MacBook Pro (2.2 GHz Intel Core i7/16 GB 1600 MHz DDR3)`编译一个半小时。在编译过程中可能会有因为各种问题而中断编译。在文章[Android源码学习（二）：编译问题总结](http://blog.alexwan1989.com/2016/03/04/Android%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0%EF%BC%9A%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E4%B8%AD%E7%9A%84%E9%97%AE%E9%A2%98/)中对常遇到的问题进行了总结。
```java
**make completed successfully (xx:xx (mm:ss))**
```
如果出现以上的提示，恭喜，表示你你已经编译成功。

因为编译时选择的是aosp_arm-eng，所以用的是模拟器来进行调试
```java
$ emulator
```
模拟器启动很慢，大概10分钟这样模拟器运行成功。有资源的同学建议还是用真机来刷。具体的刷机参考[Flash a Device](https://source.android.com/source/building.html#flash-a-device)，这里就不详细说明了。
#### 四、查看源码
Mac下查看android源码，这里推荐[Sublime Text](https://www.sublimetext.com/)配合插件还是比较好用的，类似这样
<div>
<img src="http://res.cloudinary.com/dmfz9aun7/image/upload/v1457875178/android/sublime_text_android_source.jpg" style="width:650px">
</div>

具体可以参考[Android 源码Mac-OSX查看工具Sublime-Text2](http://blog.csdn.net/wangbaochu/article/details/44836661)
来安装[Package Control插件](https://packagecontrol.io/installation)和CTags Package：
- 安装Package Control插件
	1. 打开控制台：View->show console 或 ctrl + ~
	2. 本文用是Sublime Text3 , 输入以下命令
		```java
		import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
		```
- 安装CTags Package

	1. 首先打开Sublime Text3，右键 -> Preference -> Packages Browse。查看是否已经安装了CTags Package，如果没有则继续下面步骤。
	2. 右键 -> Preference -> Package control, 输入"install package"，它会找出你可以安装的插件，在列表中选择ctag插件进行安装
	3. 修改函数跳转方式: 默认函数跳转：Ctrl+shift+左键; 跳转返回：Ctrl+shift+右键。修改方法：
		```java
		Perference->Package Settings->CTags->Mouse Binding Default->复制全部->粘贴到Perference->Package Settings->CTags->Mouse Binding User
		```
		把里面的"ctrl+shift"，修改为“command”，这样就可以用“command+左键”跳转了
		```java
		[  
		{  
		  "button": "button1",  
		  "count": 1,  
		  "press_command": "drag_select",  
		  "modifiers": [“command”],  
		  "command": "navigate_to_definition"  
		},  
		{
		  "button": "button2",  
		  "count": 1,  
		  "modifiers": ["command"],  
		  "command": "jump_prev"  
		}  
		]	  
		```
- 导入Android 源码工程

	1. 在Sublime Text3工具栏点击 Project->Open Project, 选择Android源码根目录作为工程导入
	2. 右键点击Side Bar中android 源码根目录，右键-> CTgas: Rebuild Tags, 创建索引
	3. 接下来就利用快捷键浏览代码了：
		```java
		Command+P：查找文件
		Command+R：查找方法
		Command+左键：文件或函数跳转        
		Command+右键：返回文件或函数跳转的原始位置
		```
