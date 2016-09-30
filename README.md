BintrayUploadGradle
===

Used for bintrayUpload repo gradle scripts. Then yon can add your repo in JCenter.


Details view my blog: [如何把Android Studio项目发布到中央仓库JCenter/第三方仓库JitPack中
](https://isanwenyu.github.io/2016/09/29/gradle-push-repo-maven/)

# Android仓库

> 简单的普及下关于`Android`的依赖仓库，有两种分别是`JCenter`与`Maven Central`其实不管是`JCenter`还是`Maven Central`都是`Maven`库。

## JCenter

`JCenter`是由`bintray.com`维护，在`Android Studio`的项目根目录的`build.gradle`中我们会看到自动帮我们实现的`JCenter`

```
buildscript {
    repositories {
        JCenter()
    }
}
```
## Maven Central

当然也可以在`build.gradle`中定义`Maven Central`
	
```
buildscript {
    repositories {
        mavenCentral()
    }
}
```

至于在`Android Studio`中为什么默认使用`JCenter`原因还是有的，感兴趣的可以自己去查，我要说的一点就是，这里你可以认为`JCenter`是`Maven Central`的超集，这样就能更好的理解为什么要使用`JCenter`了。
## gradle获取library

这里要了解一下我们看到的依赖所定义的方式，其实是有格式的并不是随便乱写的。其实你只要平常够仔细就能发现他们的格式是一样的。
由`GroupId`、`ArtifactId`与`VersionId`组成。例如`com.jakewharton:butterknife:6.1.0`它的GroupId是`com.jakewharton，ArtifactId`是`butterknife`，`VersionId`是`6.1.0`。现在看这些依赖是不是更清晰了呢。当我们添加了依赖之后`gradle`会先去`Maven`中查找是否有该library如果有就会使用上面定义的形式下载`http://JCenter.bintray.com/GroupId/ArtifactId/VersionId`

`http://JCenter.bintray.com/com/jakewharton/butterknife/6.1.0`
通过该链接下载到本地再与我们的项目结合。

> **下面正式进行实现依赖的实现**
    
# 使用gradle发布项目到JCenter中央仓库

## 注册bintray

首先要在[https://bintray.com](https://bintray.com) 中注册账号，注册都是很简单的就不所说了。
然后回到主页在你的`Owned Repositories`中看下你是否已经添加了`maven`。
图
第一次注册的应该没有，所以我们要先`New Repository`创建`maven`

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%884.31.14.png" height="480"/>

创建之后会自动跳转到`maven`中，你会发现没有`package`,我们可以`Add New Package`这种相信都会，我这里要说的是另外一种，我们直接在`Android Studio`中进行配置然后上传到`Bintray`。

## 代码中配置

### 分离成多个Module

为了使别人能更好的使用，我们一般在实现自己的依赖的时候会把实现的该依赖的源码作为一个`Module`,再把实例代码作为`Application Module`。所以我们要事先处理好`Module`，下面是我弄好的示例

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%884.37.33.png" width="240px"/>


### 添加bintray插件


在分了`Module`之后，首先在项目的根目录的`build.gradle`的
`dependencies`中添加`bintray`插件

```
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3
```

### 添加bintray认证

找到local.properties文件在其中添加

```
bintray.user=xxxx
bintray.apikey=xxx
```
`bintray.user`是注册的`user`，至于`bintray.apikey`在J`Frog Bintray`中的`Your Profile`的`Edit`页面的`API Key`中能找到。

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%884.41.09.png" width="240px"/>
<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%884.43.30.png" width="480px"/>


### 修改Module中的build.gradle

在其中添加如下示例代码：	

```
ext {
    bintrayRepo = 'maven'
    bintrayName = 'LazyFragment'
    publishedGroupId = 'com.isanwenyu.lazyfragment'
    libraryName = 'LazyFragment'
    artifact = 'lazyfragment'
    libraryDescription = 'Android imitation WeChat lazy loading fragments'
    siteUrl = 'https://github.com/isanwenyu/LazyFragment'
    gitUrl = 'https://github.com/isanwenyu/LazyFragment.git'
    libraryVersion = '1.0.0'
    developerId = 'isanwenyu'
    developerName = 'isanwenyu'
    developerEmail = 'isanwenyu@163.com'
    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]
}
```

同时在最后添加两个脚本

	
```
apply from: 'https://raw.githubusercontent.com/isanwenyu/BintrayUploadGradle/master/bintray_upload.gradle'
```
这是我这个依赖的示例。其中`bintrayRepo`是默认的使用`maven`，`lazyfragment`是建立的`package name`，`siteUrl`是你的项目地址，我的项目在`github`上，所以是`github`项目的地址形式。`gitUrl`是`VCS`。其他应该没什么问题，相应的改成自己的信息。

这样就构建好了依赖`com.isanwenyu.lazyfragment:lazyfragment:1.0.0`

## 上传到bintray

### 打开`Android Studio`的终端

- 编译library文件

在终端输入

```	
./gradlew install
```
出现`BUILD SUCCESSFUL`就没问题

- 上传

在终端输入

```
./gradlew bintrayUpload
```

### 或直接通过gradle面板 依次双击执行`build/assemble` `publishing/bintrayUpload`

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%884.50.15.png" width="240px" />

#### 注意： 
  在工程根目录下的build.gradle中
  
   ```
   allprojects {
    repositories {
        jcenter()
    }
    //解决android support包找不到问题
    tasks.withType(Javadoc).all { enabled = false }
}
   ```
    
一样的出现**BUILD SUCCESSFUL**就没问题
然后你进入`JFrog Bintray`进入`maven`你就会发现其中多了一个`package`，如果有的话那就表示完美成功。

## 同步到JCenter

完成了上面的步骤并不代表别人可以直接使用

```
dependencies {
    compile 'com.isanwenyu.lazyfragment:lazyfragment:1.0.0'
}
```
还要在工程根目录下的`build.gradle`中添加

```
allprojects {
    repositories {
        jcenter()
        maven {
            url 'https://dl.bintray.com/isanwenyu/maven/'
        }
    }
}
```
所以我们要同步到`JCenter`中，怎么同步呢？别急，只要在你刚刚生成的`package`点击`Add to JCenter`即可。


进去之后直接发送就可以，不需要填写什么，直接send就可以了。


最后就是等待了。几个小时之后你会收到考核通过的消息，再返回`package`就会发现`Linked to`发生了变化。

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-30%20%E4%B8%8B%E5%8D%882.50.09.png" width="480px"/>

现在你也可以通过[`http://JCenter.bintray.com/com/isanwenyu/LazyFragment/lazyfragment/1.0.0`](http://JCenter.bintray.com/com/isanwenyu/LazyFragment/lazyfragment/1.0.0) 查看。


同时别人也能使用你的依赖，通过如下简单的配置

```
dependencies {
    compile 'com.isanwenyu.lazyfragment.lazyfragment:1.0.0'
}
```

## 版本更新

对于版本更新，只要更改上面配置的版本号`libraryVersion = '1.1.0'`改成你要更新的版本，其它不变。再重新上传到`bintray`。

## 总结

整理上传流程如下：

   * 既然要上传到`JCenter`上，自然要去`https://bintray.com` 中注册账号
   * 根据自己的需求创建`maven`的`Repository`
   * 把项目分离成`Library Module`   
   * 在`local.properties`中添加`bintray.user`及`bintray.apikey`
   * 修改`Module`中的`build.gradle`中的配置
   * 在`Android Studio`终端使用`./gradlew xxx`上传
   * 最后在`JFrog Bintray`中同步到`JCenter`中
    
# 发布到JitPack

## 登录[JitPack](https://jitpack.io/)，可以直接通过`GitHub`授权登陆
   
## 在编辑框中输入`GitHub Reop`的网址，完成后点击LookUp 

<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%883.57.51.png" width="480px" alt="img"/>
### 添加`JitPack`仓库到根目录的`build.gradle`中 
	
	添加到`repositories`的尾部：
	
	```
	allprojects {
		repositories {
			...
			maven { url "https://jitpack.io" }
		}
	}
	```
### 添加引用
	
	```
	dependencies {
	        compile 'com.github.isanwenyu:LazyFragment:1.0.0'
	}
	```

## 注意：
1. 上传到github的版本需要有一个`Release`，设置`Tag`
2. 也可以直接通过`commitid`获取
	
<img src="http://oda2i06ov.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-29%20%E4%B8%8B%E5%8D%883.53.42.png" width="480px" alt="img"/>
   
# 引用
1. [如何使Android Studio项目发布到JCenter中](http://idisfkj.github.io/2016/06/13/%E5%A6%82%E4%BD%95%E4%BD%BFAndroid-Studio%E9%A1%B9%E7%9B%AE%E5%8F%91%E5%B8%83%E5%88%B0JCenter%E4%B8%AD/)

2. [发布自己项目让别人可以在dependencies中compile的更简单](http://blog.csdn.net/jonstank2013/article/details/50519628)


