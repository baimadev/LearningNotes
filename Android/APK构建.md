
## 签名

### 签名作用

1，确定发布者的身份，确定发布者的身份信息。

2，确保应用的完整性，签名会对应用程序中的所有文件进行保护，从而确保应用程序中的所有文件不会被替换。

## v1和v2的区别

### v1

签名完之后是META-INF 目录下的三个文件：MANIFEST.MF、CERT.SF、CERT.RSA。  

- MANIFEST.MF ：存放apk各个文件名称和摘要
- CERT.SF ：对MANIFEST.MF的摘要
- CERT.RSA ： 用私钥对CERT.SF文件进行签名，然后将公钥和数字证书一同写入CERT.RSA  

v1：APK v1的缺点就是META-INF目录下的文件并不在校验范围内，所以之前多渠道打包等都是通过在这个目录下添加文件来实现的。

v2: 对整个APK包文件进行了签名，签名信息不再以文件的形式存储，而是转成二进制直接写在apk文件中。  

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
