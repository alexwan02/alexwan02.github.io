---
title: Android NDK开发（二）：使用Android Studio开发进行NDK开发
date: 2016-04-18 11:32:17
tags:
---
[Android NDK Preview](http://tools.android.com/tech-docs/android-ndk-preview)

[Experimental Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/gradle-experimental)

[NDK-JNI实战教程（四）再谈新工具及NDK开发调试](http://blog.csdn.net/yanbober/article/details/51027520)

[使用AndroidStudio进行NDK开发（一）](https://dailyios.com/article/Using_Android_Studio_for_NDK_development.html)

### 一、Android Studio环境配置
#### 1、介绍
Android Studio的插件是基于Gradle的新组件模型机制，减少了NDK环境配置时间。同时含有编译JNI应用所需的NDK集成环境。本篇用户指导详细介绍如何使用Android Studio的新插件、标出新的插件和传统插件的区别。
> 注意：新的插件还是预览版，对应的Gradle api不是最终的版本，也意味着不同插件需要配合对应版本的Gradle来使用，DSL也可能变动。

#### 2、最新版本
Check [https://bintray.com/android/android-tools/com.android.tools.build.gradle-experimental/view](bintray repository) 查看最新版本；
#### 3、配置
- Gradle（根据下面的表格选择对应的插件版本）
- Android NDK r10e（如需使用NDK）
- SDK的Build Tools 至少需要19.0.0 ，意图在未来的版本变动最小化迁移过流程中带来的变动。一些特性需要更新的版本。
-  每个不同版本的插件需要配置对应的Gradle版本

	{% codeblock lang:java %}
	┌───────────┬────────────┐
	    Plugin Version  	    Gradle Version          	
	├───────────┼────────────┤
	    0.1.0                   2.5                            	
	├───────────┼────────────┤
	    0.2.0                   2.6 	                          	
	├───────────┼────────────┤
	    0.3.0-alpha3            2.8                           	
	├───────────┼────────────┤
	    0.6.0-alpha1            2.8                            	
	├───────────┼────────────┤
	    0.6.0-alpha5            2.10                           	
	├───────────┼────────────┤
	    0.7.0-alpha1            2.10                           	
	└───────────┴────────────┘
	{% endcodeblock %}
#### 4、从传统Android Gradle 插件迁移
经典的Android Studio项目类似下面的目录结构，迁移新的ndk插件需要修改标出的文件内容。传统插件和新插件的有几个重要的DSL内容变化。
{% codeblock lang:java %}
.
├── app/
│   ├── app.iml
│   ├── build.gradle  ◀
│   └── src/
├── build.gradle ◀
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties ◀
├── gradle.properties
├── gradlew*
├── gradlew.bat
├── local.properties
├── MyApplication.iml
└── settings.gradle
{% endcodeblock %}

##### （1）./gradle/wrapper/gradle-wrapper.properties 
- 新插件每个版本支持指定的gradle版本。参考[Gradle Requirements](http://tools.android.com/tech-docs/new-build-system/gradle-experimental?pli=1#TOC-Gradle-Requirements) 选择Gradle。

```java
#Wed Apr 10 15:27:10 PDT 2013
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
```

##### （2）./build.gradle
- 插件路径 `com.android.tools.build:gradle-experimental` 替换 `com.android.tools.build:gradle`.

```java
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath "com.android.tools.build:gradle-experimental:0.7.0-alpha4"
    // NOTE: Do not place your application dependencies here; they belong
    // in the individual module build.gradle files
  }
}

allprojects {
    repositories {
        jcenter()
    }
}
```
##### （3）./app/build.gradle
DSL有意义的变化，我们知道很多变化看起来没有必要的，而我们的目的是移除一些这些变化，来减少在未来处理迁移的处理。
> 注意：从版本`0.6.0-alpha5`开始，DSL有部分重要的提升。例子中的代码不适应之前版本，如果你使用旧版本插件，请参考[旧版本用户指导](https://sites.google.com/a/android.com/tools/tech-docs/new-build-system/gradle-experimental/0-4-0)

DSL 变化：

- 插件名称由`com.android.application`变为`com.android.model.application`，或者用`apply plugin: "com.android.model.library" ` 创建Android aar Library。
- 用`model { } `来包裹配置。
- 用add方法来为集合添加元素。

当前DSL限制 

- 属性列表只能设置它们的直系类型，无法接受其他类型。例如：
	- 你可以用String来设置File类型的属性，但List<File>只能接受文件类型
- 创建编译类型或产品需要调用create方法，修改已经存在的发布或编译类型可以使用名字来完成。
- 修改变量和任务的DSL很受限制。
  {% codeblock lang:java %}
  apply plugin: "com.android.model.application"
  model {
    android {
      compileSdkVersion 23
      buildToolsVersion "23.0.2"

      defaultConfig {
        applicationId "com.example.user.myapplication"
        minSdkVersion.apiLevel 15
        targetSdkVersion.apiLevel 22
        versionCode 1
        versionName "1.0"

        buildConfigFields {
          create() {
            type "int"
            name "VALUE"
            value "1"
          }
        }
      }
      buildTypes {
        release {
          minifyEnabled false
          proguardFiles.add(file("proguard-rules.pro"))
        }
      }
      productFlavors {
        create("flavor1") {
          applicationId "com.app"
        }
      }
	    
      // Configures source set directory.
      sources {
        main {
          java {
            source {
              srcDir "src"
            }
          }
        }
      }
    }
  }

  dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:22.2.0"
  }
  {% endcodeblock %}
##### （4）签名配置
用`$()`语法指定其他的model元素。使用`$()`，需要为Gradle 2.10以下版本添加`"-Dorg.gradle.model.dsl=true" `Gradle命令行的参数。对指定签名配置文件很有用。
> 注意：android.signingConfigs必须在`android {} `块之外。

	```java
	apply plugin: "com.android.model.application"
	
	model {
	  android {
	    compileSdkVersion 23
	    buildToolsVersion "23.0.2"
	    buildTypes {
	      release {
	        signingConfig = $("android.signingConfigs.myConfig")
	      }
	    }
	  }
	  android.signingConfigs {
	    create("myConfig") {
	      storeFile "/path/to/debug.keystore"
	      storePassword "android"
	      keyAlias "androiddebugkey"
	      keyPassword "android"
	      storeType "jks"
	    }
	  }
	}
	```
#### 5、NDK集成环境

新的插件集成了NDK开发环境，支持创建Native应用。如何使用NDK集成环境：
- 用Studio内部SDK Manager 下载 NDK
- 设定local.properties中的ndk.dir 或ANDROID_NDK_HOME环境变量的值，为NDK的路径
- 在build.gradle 中添加`android.ndk`模块
##### （1）简单NDK配置
一个简单NDK应用的build.gradle如下：
```java
apply plugin: 'com.android.model.application'

model {
  android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    ndk {
      moduleName "native"
    }
  }
}
```
> 注意：`moduleName`参数不能省略，决定native库的名称

##### （2）Ndk Source 设置
默认情况下，会查找` src/main/jni`中的 C/C++ 文件。配置android.sources 修改源码路径
```java
model {
  android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    ndk {
      moduleName "native"
    }
    sources {
      main {
        jni {
          source {
            srcDir "src"
          }
        }
      }
    }
  }
}
```
JNI源码包含C和C++ 文件，子目录中所有文件都包含。带有.c后缀的文件为 C 文件，而C++ 文件则有以下几种后缀：.C ，.CPP，.c++ .cp ，.cpp ，.cxx。可以使用`exculde`方法排除文件，用`include`忽略文件。
```java
model { 
  android.sources {
    main {
      jni {
        source {
          include "someFile.txt"
          exclude "**/excludeThisFile.c"
        }
      }		
    }
  }
}
```
##### （3）其他编译参数
可以使用`android.ndk{}`块来设置参数
```java
model {
  android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    ndk {
      // All Configurations that can be changed in android.ndk
      moduleName "native"
      toolchain "clang"
      toolchainVersion "3.5"
      // Note that CFlags has a capital C ,which is inconsistent with 
      // the naming convention of other properties. This is a 
      // technical limitation that will be resolved
      CFlags.add("-DCUSTOM_DEFINE")
      cppFlags.add("-DCUSTOM_DEFINE")
      ldFlags.add("-L/custom/lib/path")
      ldLibs.add("log")
      stl "stlport_static"
    }
    buildTypes {
      release {
        ndk {
          debuggable true
        }
      }
    }
    productFlavors {
      create("arm") {
        ndk {
          // You can customize the NDK configurations for each
          // productFlavors and buildTypes.
          abiFilters.add("armeabi-v7a")	  
        }
      }
      create("fat") {
        // If ndk.abiFilters is not configured, the application
        // compile and package all suppported ABI.
      }
    }
  }
  // You can modify the NDK configuration for each variant.
  components.android {
    binaries.afterEach { binary ->
      binary.mergedNdkConfig.cppFlags.add( "-DVARIANT=\"" + binary.name + "\"")
    }
  }
}
```

##### （4）已知限制
1. 不支持类似cpu_feature的NDK 单元
2. 不支持合并外部build系统 

##### （5）示例
访问Github的[Ndk Samples](https://github.com/googlesamples/android-ndk)

##### （6）多重NDK Project
0.4.0 Plugin 添加了NDK依赖的基础支持，可以创建一个native 库。如果使用0.4.0 Plugin可以用Gradle编译native项目，但在Android Studio中编辑和调试功能尚未实现。

- 独立NDK Plugin

	在gradle-experimental:0.4.0中，一个新的Plugin可以只创建native library，无须创建android application 或 android 	library。DSL与 application/library Plugin相近。下面的例子中`build.gradle`可以使用`src/main/jni`的C/C++源文件来生成`libhello.so`文件
	
	```java
	apply plugin: "com.android.model.native"
	model {
	  android{
	    compileSdkVersion 23
	    ndk {
	      moduleName "hello"
	    }
	  }
	}
	
	```
- 已知问题
	- Android Studio 未支持编辑单独plugin
	- 编译application时修改library源文件后，不会自动重链接到新的library

##### （6）Ndk依赖
指定依赖的语法遵照Gradle未来依赖系统的方式，你可以设定依赖的Android Project或指定文件。
比如，假设有个使用独立NDK Plugin的subproject

{% codeblock lang:java lib/build.gradle %}
apply plugin: "com.android.model.native"
model {
  android{
    compileSdkVersion 23
    ndk {
      moduleName "hello"
    }
    sources {
      main {
        jni {
          exportedHeaders {
            srcDir "src/main/headers"
          }
        }
      }
    }
  }
}
{% endcodeblock %}

任何带有JNI依赖的项目需要包含exportedHeaders指定的目录。你可以为项目的依赖项目添加JNI代码

{% codeblock lang:java app/build.gradle %}
apply plugin: "com.android.model.application"
model{
  android{
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    source {
      main {
        jni {
          dependencies {
            project ":lib1"
          }
        }
      }
    }
  }
}
{% endcodeblock %}

你可以指定target项目的buildType 和（或）productFlavor 。否则，plugin会查找相同的buildType和productFlavor作为你的application。你也可以指定linkageType，如果希望native library最为静态链接库的话。

```java
model{
  android.sources {
    main {
      jni {
        dependencies {
          project ":lib1"
          buildType "debug"
          productFlavor "flavor1"
          linkage "static"
        }
      }
    }
  }
}
```
声明依赖文件，创建预编译库，添加依赖库等

{% codeblock lang:java %}
model {
  repositories {
    libs (PrebuiltLibraries) {
      prebuilt {
        headers.srcDir "path/to/headers"
        binaries.withType (SharedLibraryBinary) {
          sharedLibraryFile = file("lib/${targetPlatform.getName() }/prebuilt.so")
        }
      }
    }
  }
  android.sources {
    main {
      jniLibs {
        dependencies {
          library "prebuilt"
        }
      }
    }
  }
}

{% endcodeblock %}

可以添加native 依赖到"jniLibs"或"jni" source 集合中。当添加natice依赖库到"jniLibs"中，依赖会打包到` application/library`下，但不会用来编译JNI代码。
```java
model {
  android.sources {
    main {
      jniLibs {
      	dependencies {
          library "prebuilt"
        }
      }
    }
  }
}

```

#### 6、DSL变化
Plugin还在试验阶段，DSL会随着plugin开发版本变动。本节来说明不同版本的DSL变化，来帮助迁移。

##### （1）、0.6.0-alpha1 -> 0.6.0-alpha5
- Plugin需要gradle 2.10 ，带来DSL签名方面的改善
- 配置信息可以被折叠

	```java
	android{
	  buildTypes {
	   ... 
	  }
	}
	```
	替换
	
	```java
	android.buildTypes {
	  ...
	}
	
	```
- 文件类型接受String，但String无法添加到List<File>中
- -Dorg.gradle.model=true 现在为默认值。允许被其他model引用，但是引用的model必须为独立的block
- 大多数参数不在需要'='来设值

##### （2）、0.4.x -> 0.6.0-alpha1
- 指定依赖指定库文件的DSL修改为遵守Gradle的native依赖DSL。[Sample](https://github.com/gradle/gradle/blob/master/subprojects/docs/src/samples/native-binaries/prebuilt/build.gradle)

	{% codeblock lang:java %}
	model {
	  repositories {
	    prebuilt(PrebuildtLibraries) {
	      binaries.withType(SharedLibraryBinary){
	        sharedLibraryFile = file("lib/${targetPlatform.getName()}/prebuilt.so")
	      }
	    }
	    android.sources {
	      main{
	        jniLibs {
	          dependencies {
	            library "prebuilt"
	          }
	        }
	      }
	    } 
	  }
	}
	
	{% endcodeblock%}

##### （3）、0.2.x -> 0.4.0
- += 不在用于collections。列表添加Items可以使用'add'和'addAll'放。如 `CFlags += "-DCUSTOM_DEFINE"`用`CFlags.add("-DCUSTOM_DEFINE")`替换。

##### （4）、0.1.x -> 0.2.x
- jniDebuggable 从buildType 移动到ndk模块中
	```java
	release {
	  jniDebuggable = true
	}
	```
	变为
	```java
	release {
	  ndk.with {
	    debuggable = true
	  }
	}
	```