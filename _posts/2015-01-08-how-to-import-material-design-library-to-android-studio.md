---
title: 如何在 Android Studio 中引入MaterialDesign Library
tags: 技术 Android
---  

[MaterialDesignLibrary](https://github.com/navasmdc/MaterialDesignLibrary) 开源库实现了在 Android Material 设计中提到的一些组件。在引入到Android Studio项目的时候，碰到几个问题，一并记录下来。
<!--more-->
从github将整个MaterialDesignLibrary下来之后，可以看到他专门有一个`AndroidStudio/MaterialDesign` 目录。

###步骤：

1. 将`AndroidStudio`下的`MaterialDesign`目录，放到项目的里和`app`同级别的目录当中。或者在`File->Import Moudle` 将其导入成模块也可以。
2. 如果你使用的Gradle版本在0.14.0（2014/10/31）以上，那需要编辑MaterialDesign目录下面的build.gladle 文件，将 `runProguard` 改为 `minifyEnabled`。 [^1]
3. 修改`MaterialDesign/build.gladle` 文件,:
    -  将 `apply plugin: 'com.android.application'` 改成 `apply plugin: 'com.android.library'`
    -  将 `applyicationId` 去掉 
    -  把 miniSDK 改成和项目一致的版本
4. 在settings.gradle 里面include加上 `':MaterialDesign'`
5. 在自己的项目文件的 build.gradle 中加入一行：`compile project(':MaterialDesign')`

###可能遇到的问题

##### Error:Library projects cannot set applicationId. 
Library的项目不能设置ApplicationID的，所以，删掉就好了。

##### Error:(15, 0) Gradle DSL method not found: 'runProguard()' 
Gradle现在的版本把runProguard 改叫 minifyEnabled了。

### 参考
1. [How To Import Material Design Library To Android Studio](http://stackoverflow.com/questions/27364565/how-to-import-material-design-library-to-android-studio)
2. [After Android Studio update: Gradle DSL method not found: 'runProguard()'](http://stackoverflow.com/questions/27343745/after-android-studio-update-gradle-dsl-method-not-found-runproguard)
3. [Android Studio 简介及导入 jar 包和第三方开源库方法](http://drakeet.me/android-studio)
4. [导入开源库到基于Android Studio构建的项目中](http://blog.isming.me/2014/12/12/import-library-to-android-studio/)

[^1]: 相关更新说明可见Changelog：http://tools.android.com/tech-docs/new-build-system 