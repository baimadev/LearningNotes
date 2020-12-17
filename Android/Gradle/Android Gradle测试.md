# Android Gradle测试

```java

    defaultConfig {
       ...
        testApplicationId "com.baima.jetpack.test"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        testHandleProfiling true //是否启用性能分析
        testFunctionalTest true //是否启用功能测试
    }
```

//最后会根据配置生成AndroidManifest文件


```java

android{

testBuildType 'debug'//将测试包类型设为debug，默认就是debug

}

```

## 1.本地单元测试

直接运行在本地开发机器上，不依赖Android框架。    
使用JUnit框架进行测试  
Context可以借助一些模拟框架来依赖这种关系，如Mockito和JMock。  
如果要最大限度的运用本地单元测试，最重要的是分好层级，降低代码和Android之间的耦合程度，比如MVP。

执行测试：`gradlew :app:test`

## 2.Instrument测试
可以使用Android SDK框架所有类和特性。

```java
android {
    defaultConfig {
       ...

        testApplicationId "com.baima.jetpack.test"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        testHandleProfiling true //是否启用性能分析
        testFunctionalTest true //是否启用功能测试
    }

}

dependencies {
...
    androidTestImplementation 'androidx.test.ext:junit:1.1.2'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.3.0'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test:rules:1.0.2'
}

```

执行测试：`gradlew connectedAndroidTest`，在build/report生成测试报告。

### testOptions

```java

  android{...}
  testOptions{
        reportDir = "$project.buildDir/app/report" //生成的测试报告目录
        resultstDir = "$project.buildDir/app/result"//生成测试结果的目录
        unitTests.all{
            jvmArgs '-XX:MaxPermSize = 256m' //对所有的Test任务做操作
                                             //指定启动的JVM最大非堆内存是256M
        }
    }
```

### 代码覆盖率

```java
buildTypes {
		 ...

        debug{
            testCoverageEnabled = true
        }
    }

```

testCoverageEnabled用于控制代码覆盖率统计是否开启，会生成代码覆盖率的报告，用来补全测试用例。生成报告的目录： **app/build/reports/coverage**

## 3.Lint支持

Android为我们提供的针对代码、资源优化的工具，它可以帮助我们检查哪些资源没有被使用，哪些使用了新的API，哪些资源没有国际化等。它会生成一份报告在**out/lint-results.html**下。

执行lint检查：`gradlew lint` 

```java
android{
	lintOptions{
        abortOnError true //遇到错误停止构建
        warningAsErrors true //将waring当成错误处理
        check 'NewApi' //接收一个issue id作文参数，开启对issue的检查
        disable ‘issue id’ //关闭对某issue的检查
        //lint --list可查看所有可用的id
        
    }
}

```
