## ContentProvider
进程间进行数据交互 & 共享，即跨进程通信。    
其数据源可以是数据库、xml、网络、文件等。    
底层采用的是Binder机制。  

## 基本使用

外界进程通过 URI（统一资源标识符） 找到对应的ContentProvider其中的数据，再进行数据操作。

URI = Scheme://Authority/Path/ID  
content://com.carson.provider/User/1

### ContentProvider类

- 以表格的形式组织数据，提供增删改查API。    
- Android为常见的数据（如通讯录、相册等）提供了内置了默认的ContentProvider。  
- ContentProvider类并不会直接与外部进程交互，而是通过ContentResolver 类。

### ContentResolver类

ContentResolver类对所有的ContentProvider进行统一管理。  

ContentResolver 类提供了与ContentProvider类相同名字 & 作用的4个方法（增删改查）。

实例说明：

```java

ContentResolver resolver =  getContentResolver();

Uri uri = Uri.parse("content://cn.scu.myprovider/user");

Cursor cursor = resolver.query(uri, null, null, null, "userid desc");

```

### 工具类

- ContentUris：操作URI
- UriMatcher：在ContentProvider中注册URI、根据URI匹配ContentProvider中对应的数据表。
- ContentObserver：观察 Uri引起 ContentProvider 中的数据变化 & 通知外界（即访问该数据访问者）
