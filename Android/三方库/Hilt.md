# Hilt

**优点：管理实例的生命周期，只能在对应的范围内进行使用。会自动释放不在使用的对象，减少资源的过度使用。**

##

```java
//project 的 build.gradle 添加以下依赖。
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}


//App 模块中的 build.gradle
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
}

//Hilt 使用 Java 8 的功能，所以要在项目中启用 Java 8，需要在 App 模块的 build.gradle 文件中，添加以下代码

android {
  ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    // For Kotlin projects
    kotlinOptions {
        jvmTarget = "1.8"
    }
}


```

## 不使用Module的注入

```kotlin
@HiltAndroidApp
class MyApplication : Application() {
}
```

```kotlin
class ServiceTest @Inject constructor(){
    fun hiltTest(){
        Log.e("xia"," hilt test")
    }
}
```
### 使用
- 如果使用 @AndroidEntryPoint 注解 Android 类，还必须注解依赖他的 Android类，例如： 给 fragment 使用 @AndroidEntryPoint 后，则还需要给 fragmet 依赖的 Activity 依赖 @AndroidEntryPoint ，否则会出现异常

- @AndroidEntryPoint 注解 仅仅支持 ComponentActivity 的子类，例如 AppCompatActivity ；Fragment：仅仅支持继承 androidx.Fragment 的 Fragment。

- Hilt 会根据相应的 Android 类生命周期自动创建和销毁组件的实例。

-  Hilt注入的字段不能为private。

```kotlin

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var stest: ServiceTest

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        stest.hiltTest()
    }
}
```

## 使用Module的注入

- @Module 模块用于向 Hilt 添加绑定，告诉 Hilt 如果提供不同类型的实例。
使用了 @Module 的类，相当于是一个模块，常用于创建依赖对象（如，Okhttp，Retrofit 等）。

- 使用 @Module 的类，需要使用 #InstallIn 指定此 module 的范围，会绑定到对应 Android 类的生命周期上

| Hilt 提供的组件  | 对应的 Android 类的活动范围|
|  ----  | ----  |
| ApplicationComponent  | Application|
| ActivityRetainedComponent  | ViewModel |
|  ActivityComponent | Activity|
|  FragmentComponent | Fragment|
|  ViewComponent | View|
|  ViewWithFragmentComponent | View annotated with @WithFragmentBindings|
|  ServiceComponent | Service|

**Hilt 会根据相应的 Android 类生命周期自动创建和销毁生成的组件类的实例，它们的对应关系如下表格所示。**  
**使用@Single时，必须使用ApplicationComponent，因为它才对应全局。**  

| Hilt |提供的组件创建对应的生命周期| 销毁对应的生命周期|
| ---- | ---- | ---- |
|ApplicationComponent|Application#onCreate()|Application#onDestroy()|
|ActivityRetainedComponent|Activity#onCreate()|Activity#onDestroy()|
|ActivityComponent|Activity#onCreate()|Activity#onDestroy()|
|FragmentComponent|Fragment#onAttach()|Fragment#onDestroy()|
|ViewComponent|View#super()|View destroyed|
|ViewWithFragmentComponent|View#super()|View destroyed|
ServiceComponent|Service#onCreate()|Service#onDestroy()|


- @Providers，常用于被 @Module 注解标记类的内部方法，并提供依赖项对象。

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object AnalyticsModule{

    //每次都是新的
    @Provides
    fun bindHiltTest():DataDao{
        return  DataDao("ahaha")
    }

    //单例
    @Provides
    @Singleton
    fun bindOkTest():Test{
        return  Test("OK")
    }
}
```
### 注入第三方库

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object RoomModel {

    /**
     * @Provides：常用于被 @Module 标记类的内部方法，并提供依赖对象
     * @Singleton：提供单例
     */
    @Provides
    @Singleton
    fun provideAppDataBase(@ApplicationContext application: Application): AppDataBase {
        return Room
            .databaseBuilder(application, AppDataBase::class.java, "knif.db")
            .fallbackToDestructiveMigration()
            .allowMainThreadQueries()
            .build()
    }

    @Provides
    @Singleton
    fun providerUserDao(appDataBase: AppDataBase): UserDao {
        return appDataBase.getUserDao()
    }
}
```
**application根据InstallIn选择的Android类，自动导入参数**

**调用providerUserDao时会自动调用上方的provideAppDataBase来获取dataBase,
因为provideAppDataBase也是用了 @Provides注解。**

### 使用binds注入接口
Binds：必须注释一个抽象函数，抽象函数的返回值是实现的接口。通过添加具有接口实现类型的唯一参数来指定实现。

```kotlin
interface User{
    fun getName():String
}

class UserImpl @Inject constructor():User{
    override fun getName(): String {
        return "xiarupeng"
    }

}

@Module
@InstallIn(ApplicationComponent::class)
abstract class UserModule {
    @Binds
    abstract fun getUser(userImpl: UserImpl): User
}

```
**注意：这个 Module 是抽象的。**

#### 使用 @Qualifier 提供同一接口，不同的实现

```kotlin
interface User{
    fun getName():String
}

class UserImplA @Inject constructor():User{
    override fun getName(): String {
        return "xiarupeng"
    }

}

class UserImplB @Inject constructor():User{
    override fun getName(): String {
        return "xiarupengB"
    }
}
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class A

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class B

@Module
@InstallIn(ApplicationComponent::class)
abstract class UserModule {
    @Binds
    @Singleton
    @A
    abstract fun getUserA(userImpl: UserImplA): User

    @Binds
    @Singleton
    @B
    abstract fun getUserB(userImpl: UserImplB): User
}

```

使用

```kotlin
    @Inject
    @A
    lateinit var userA:User

    @Inject
    @B
    lateinit var userB:User
```

### 预定义限定符

Hilt 提供了一些预定义限定符，例如你可能在不同的情况下需要不同的 Context（Appliction、Activity）Hilt 提供了 @ApplicationContext 和 @ActivityContext 两种限定符。

### 组件作用域@scopes

@scopes 的作用在指定作用域范围内(Application、Activity 等等) 提供相同的实例。

| Android class |Generated component| Scope|
| ---- | ---- | ---- |
|Application|ApplicationComponent|@Singleton|
|ViewModel|ActivityRetainedComponent|@ActivityRetainedScope|
|Activity|ActivityComponent|@ActivityScoped|
|Fragment|FragmentComponent|@FragmentScoped|
|View|ViewComponent|@ViewScoped|
|View annotated with@WithFragmentBindings|ViewWithFragmentComponent|@ViewScoped|
|Service|ServiceComponent|@ServiceScoped|
