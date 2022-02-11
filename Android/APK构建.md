
## 签名

### 签名作用

1，确定发布者的身份，确定发布者的身份信息。

2，确保应用的完整性，签名会对应用程序中的所有文件进行保护，从而确保应用程序中的所有文件不会被替换。

## v1和v2的区别

v1：只对未压缩的文件内容进行了验证，所以在APK签名之后可以进行很多修改——文件可以移动，甚至可以重新压缩。即可以对签名后的文件再进行处理。  

v2:对整个APK包文件进行了验证。  

Android7.0只支持v1验证，如果我们需要在签名后对apk文件做处理，则应该使用v1。

## 打包

- 1）aapt 为res 目录下资源生成R.java 文件，同时为AndroidMainfest.xml 生成Mainfest.java文件  
- 2）aidl 将项目中自定义的aidl 生成相应的Java文件  
- 3）javac将项目中所有java代码编译成class文件，包括aapt生成java文件，aidl生成java文件  
- 4）混淆代码
- 5) 将所有class 文件(包括第三方class 文件) 转换为dex 文件  
- 6）通过apkbuilder工具，将aapt生成的resources.arsc和res文件、assets文件和dex文件一起打包生成apk
- 7）jarsigner 对apk 进行签名  
- 8）ziplign 对apk 进行对齐操作，以便运行时节约内存。  

![](./img/android_build.png)

R.java MANIFEST.java AIDL.java Class 混淆 dex apkbuilder 资源 dex apk 签名 对齐
