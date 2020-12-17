## 多项目设置

### 1.在settings.gradle中引入项目
```java
include ':mylibrary', ':lib', ':app'
rootProject.name = "jetpack"
```

### 2.在app的build.gradle中依赖

``` java
dependencies {
    implementation project(":lib")
    implementation project(":mylibrary")
}

```

Android Lib作为库发布时是打成一个aar包，Java Lib是打成一个jar包。  
可以设置打包的版本类型。

```java
android{
	defaultPublishConfig "flavorDebug"
}

```

## 库项目单独发布


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