# Gradle

## Java Gradle

### 常见任务

- build 编译你的源码文件、处理资源文件、打成jar包、然后编译测试用例代码、处理测试资源、最后运行单元测试

- clean 删除build目录及其他构建生成的文件

- assemble 不会执行单元测试，只会编译和打包，引导类任务，根据不同的项目类型打不同的包。

- check 只会执行单元测试 

- javadoc 为我们生成java格式的doc api文档

### 源码集合SourceSet

源集默认目录 :

- src/sourceSet name/java
- src/sourceSet name/resources

更改源集文件目录或新增源集：

```java
sourceSets{
    vip{

    }

    main{
        java{
            srcDir 'src/java'
        }
    }
}
```

### Java Gradle添加的属性

- sourceSets
- sourceCompatibility 编译java源文件使用的Java版本
- targetCompatibility 编译生成的类的Java版本
- manifest 访问或者配置我们的manifest清单文件
- libsDir 存放生成的类库目录
- distsDir 存放生成的发布的文件的目录

### 发布构件

```java


task publish(type:Jar)

artifacts {
    archives publish

}

uploadArchives{
    repositories {
        flatDir {
            name 'libs'
            dirs "$projectDir/libs"
        }
    }
}
```


### 其他
对所有子项目进行配置：

```java
subprojects {
    apply plugin: 'java'
    repositories{
        mavenCentral()
    }
    dependencies {
        testImplementation 'junit:junit:4.13.1'
    }
}
```

## Android Gradle

Android GradleJ继承于Java插件，具有所有java插件的特性

### android

- compileSdkVersion 编译所依赖的Android SDK版本

- buildToolsVersion  构建Android工程所用的构建工具的版本

- defaultConfig 它是一个ProductFlavor，可以根据不同的情况生成不同的包，例如打多渠道包

- buildTypes 是一个NamedDomainObjectContainer，相当于SourceSet

- minifyEnabled 是否开启混淆 

- proguardFiles 当我们启用混淆时，所用的proguard的配置文件 

### defaultConfig

#### 1.applicationId

用于指定生成的App的包名，默认是null，这个时候在构建的时候，会从AndroidManifest文件中读取，也就是manifest标签的package属性值。

#### 2.versionCode versionName
versionCode用于配置App的内部版本号，versionName是让用户知道我们的App版本，一个对内一个对外。

#### 3.testApplicationId
用于配置测试APP的包名，默认情况下是ApplicationId+.test。

#### 4.testInstrumentRunner
用于配置单元测试时使用的Runner，默认使用的是android.test.InstrumentationTestRunner

#### 5.signingConfig
配置默认的签名信息

```java
   //ProductFlavor
    defaultConfig {
      ...
        signingConfig signingConfigs.release
    }

    signingConfigs{
        release{
            storeFile file("Untitled.keystore")
            storePassword "xia199843"
            keyAlias "baima"
            keyPassword "xia199843"
        }
        debug{
            ...
        }
    }

    //相当于SourceSet
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }

        debug{
            ...
        }
    }

```

#### 6.proguardFile proguardFiles
用于配置App ProGuard混淆所使用的ProGuard配置文件

### BuildType


每个BuildType会生成一个SourceSet，默认位置为src//，所以针对不同对的BuildType我们可以单独为其指定java源代码、res资源等；除此之外，每一个BuildType还会生成相应的assemble任务，如assembleDebug、assembleRelease。

#### 1.applicationIdSuffix
用于配置默认ApplicationIde的后缀。

#### 2.debuggable jniDebuggable
debuggable用于配置是否生成一个可供调试的apk，jniDebuggable用于配置是否生成一个可供调试Jni（C/C++）代码的apk

#### 3.minifyEnabled proguardFile proguardFiles

```java
 proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
```

proguard-android-optimize.txt和proguard-android.txt是AndroidSDK提供的混淆配置文件。

#### 4.multiDexEnabled
用于配置该buildType是否启用自动拆分多个dex的功能。一般用程序代码太多，超过了65535个方法的时候

#### 5.shrinkResources
用于配置是否自动清理未使用的资源。

#### 6.singingConfig

#### 6.zipAlignEnabled
整理优化apk文件，提高系统和应用的运行效率，更快的读写apk中的资源，降低内存的使用。

## 任务

- connectedCheck 在所有连接设备或者模拟器上运行check检查

- deviceCheck 通过Api连接远程设备运行checks，它被用于CI（持续集成）服务器上

- lint 在所有的ProductFlavor上运行lint检查


