---
title: MAC上的Eclipse总是需要Java6运行环境？
tags: 技术 Java6 Eclipse
---  

摘要：MAC OS X更新之后，再从DOCK打开Eclipse的时候，总是会提示“你需要一个Java 6运行环境”，并给出了一个跳转到Apple的说明链接。上一次听他的装了Java 6， 这一次换一种绕过的方法。验证环境：Java 1.8.0_25, OS X EI Capitan.

<!--more-->

##### 修改方法

找到```/Library/Java/JavaVirtualMachines/jdk1.8.0_25.jdk/Contents/```

修改Info.plist，将

    <key>JVMCapabilities</key>
    <array>
        <string>CommandLine</string>
    </array>


改为：

    <key>JVMCapabilities</key>
    <array>
        <string>CommandLine</string>
        <string>JNI</string>
        <string>BundledApp</string>
        <string>WebStart</string>
        <string>Applets</string>
    </array>

然后重启电脑，不然只能双击从“应用程序”的图标启动，不能从Dock单击启动。

##### 参考
http://stackoverflow.com/questions/19563766/eclipse-kepler-for-os-x-mavericks-request-java-se-6/19594116#19594116