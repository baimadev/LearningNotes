#Gson源码解析（一）Gson序列化流程

## 我们从fromJson开始。

```java
public <T> T fromJson(String json, Class<T> classOfT) throws JsonSyntaxException {
  Object object = fromJson(json, (Type) classOfT);
  return Primitives.wrap(classOfT).cast(object);
}
```

经过几个fromJson重载方法后，会将我们传入的json一步步封装 String -> StringReader -> JsonReader；将传入的类类型Class<T>变为Type。

Note：JsonReader是流式JSON解析器。它读取文字值（字符串，数字，布尔值和空值）以及对象和数组的开始和结束分隔符，并将他们以令牌流的形式推送。

```java
public <T> T fromJson(JsonReader reader, Type typeOfT) throws JsonIOException, JsonSyntaxException {
  boolean isEmpty = true;
  boolean oldLenient = reader.isLenient();
  reader.setLenient(true);
  try {
    reader.peek();
    isEmpty = false;
    TypeToken<T> typeToken = (TypeToken<T>) TypeToken.get(typeOfT);
    TypeAdapter<T> typeAdapter = getAdapter(typeToken);
    T object = typeAdapter.read(reader);
    return object;

  ...
  //省略非关键代码
}
```

Note：这里的[TypeToken](#1)是用来在**运行时获取泛型参数的具体类型**的工具类。

可以看到最终是通过TypeAdapter.read方法来解析json。TypeAdapter从何而来？我们进入`getAdapter(TypeToken<T> type)`方法看看。

## getAdapter

```java

public <T> TypeAdapter<T> getAdapter(TypeToken<T> type) {

   //1.尝试从typeTokenCache缓存里获取TypeAdapter
   TypeAdapter<?> cached = typeTokenCache.get(type == null ? NULL_KEY_SURROGATE : type);
   if (cached != null) {
     return (TypeAdapter<T>) cached;
   }

   Map<TypeToken<?>, FutureTypeAdapter<?>> threadCalls = calls.get();
   boolean requiresThreadLocalCleanup = false;
   if (threadCalls == null) {
     threadCalls = new HashMap<TypeToken<?>, FutureTypeAdapter<?>>();
     calls.set(threadCalls);
     requiresThreadLocalCleanup = true;
   }
   
   //2.尝试从threadCalls缓存中获取TypeAdapter
   FutureTypeAdapter<T> ongoingCall = (FutureTypeAdapter<T>) threadCalls.get(type);
   if (ongoingCall != null) {
     return ongoingCall;
   }

   //3.两个缓存中都没有的话就创建一个
   try {
     FutureTypeAdapter<T> call = new FutureTypeAdapter<T>();
     threadCalls.put(type, call);

     for (TypeAdapterFactory factory : factories) {
       TypeAdapter<T> candidate = factory.create(this, type);
       if (candidate != null) {
         //存入缓存中（threadCalls和typeTokenCache）
         call.setDelegate(candidate);
         typeTokenCache.put(type, candidate);
         return candidate;
       }
     }
     throw new IllegalArgumentException("GSON (" + GsonBuildConfig.VERSION + ") cannot handle " + type);
   } finally {
     threadCalls.remove(type);

     if (requiresThreadLocalCleanup) {
       calls.remove();
     }
   }
 }
```
1.getAdapter方法首先从Gson的成员变量`Map<TypeToken<?>, TypeAdapter<?>> typeTokenCache `中查找TypeAdapter；  
2.然后在从threadCalls缓存中查找TypeAdapter，threadCalls使用了ThreadLocal存储缓存。  
`calls  = new ThreadLocal<Map<TypeToken<?>, FutureTypeAdapter<?>>>();`   
threadlocal是一个线程内部的存储类，可以在指定线程内存储数据，数据存储以后，只有指定线程可以得到存储数据。   
3.缓存中都没有的话就根据type创建一个TypeAdapter。

这里使用了两个缓存，一个存储在Gson实例的局部变量里，一个存储在线程的局部变量里。这样不管是使用同一个Gson实例解析json，还是在同一线程下使用不同Gson实例解析json都可以高效的获得TypeAdapter。

接下来我们来看看factories是怎样创建TypeAdapter的。

factories是在Gson的构造器里初始化的。

```java
Gson(...) {
     ...

   List<TypeAdapterFactory> factories = new ArrayList<TypeAdapterFactory>();

   // built-in type adapters that cannot be overridden
   factories.add(TypeAdapters.JSON_ELEMENT_FACTORY);
   factories.add(ObjectTypeAdapter.FACTORY);

   // the excluder must precede all adapters that handle user-defined types
   factories.add(excluder);

   // users' type adapters
   factories.addAll(factoriesToBeAdded);

   // type adapters for basic platform types
   factories.add(TypeAdapters.STRING_FACTORY);
   factories.add(TypeAdapters.INTEGER_FACTORY);
   factories.add(TypeAdapters.BOOLEAN_FACTORY);
   factories.add(TypeAdapters.BYTE_FACTORY);
   factories.add(TypeAdapters.SHORT_FACTORY);
   TypeAdapter<Number> longAdapter = longAdapter(longSerializationPolicy);
   factories.add(TypeAdapters.newFactory(long.class, Long.class, longAdapter));
   factories.add(TypeAdapters.newFactory(double.class, Double.class,
           doubleAdapter(serializeSpecialFloatingPointValues)));
   factories.add(TypeAdapters.newFactory(float.class, Float.class,
           floatAdapter(serializeSpecialFloatingPointValues)));
   factories.add(TypeAdapters.NUMBER_FACTORY);
   factories.add(TypeAdapters.ATOMIC_INTEGER_FACTORY);
   factories.add(TypeAdapters.ATOMIC_BOOLEAN_FACTORY);
   factories.add(TypeAdapters.newFactory(AtomicLong.class, atomicLongAdapter(longAdapter)));
   factories.add(TypeAdapters.newFactory(AtomicLongArray.class, atomicLongArrayAdapter(longAdapter)));
   factories.add(TypeAdapters.ATOMIC_INTEGER_ARRAY_FACTORY);
   factories.add(TypeAdapters.CHARACTER_FACTORY);
   factories.add(TypeAdapters.STRING_BUILDER_FACTORY);
   factories.add(TypeAdapters.STRING_BUFFER_FACTORY);
   factories.add(TypeAdapters.newFactory(BigDecimal.class, TypeAdapters.BIG_DECIMAL));
   factories.add(TypeAdapters.newFactory(BigInteger.class, TypeAdapters.BIG_INTEGER));
   factories.add(TypeAdapters.URL_FACTORY);
   factories.add(TypeAdapters.URI_FACTORY);
   factories.add(TypeAdapters.UUID_FACTORY);
   factories.add(TypeAdapters.CURRENCY_FACTORY);
   factories.add(TypeAdapters.LOCALE_FACTORY);
   factories.add(TypeAdapters.INET_ADDRESS_FACTORY);
   factories.add(TypeAdapters.BIT_SET_FACTORY);
   factories.add(DateTypeAdapter.FACTORY);
   factories.add(TypeAdapters.CALENDAR_FACTORY);
   factories.add(TimeTypeAdapter.FACTORY);
   factories.add(SqlDateTypeAdapter.FACTORY);
   factories.add(TypeAdapters.TIMESTAMP_FACTORY);
   factories.add(ArrayTypeAdapter.FACTORY);
   factories.add(TypeAdapters.CLASS_FACTORY);

   // type adapters for composite and user-defined types
   factories.add(new CollectionTypeAdapterFactory(constructorConstructor));
   factories.add(new MapTypeAdapterFactory(constructorConstructor, complexMapKeySerialization));
   this.jsonAdapterFactory = new JsonAdapterAnnotationTypeAdapterFactory(constructorConstructor);
   factories.add(jsonAdapterFactory);
   factories.add(TypeAdapters.ENUM_FACTORY);
   factories.add(new ReflectiveTypeAdapterFactory(
       constructorConstructor, fieldNamingStrategy, excluder, jsonAdapterFactory));

   this.factories = Collections.unmodifiableList(factories);
 }

```
可以看到factories是一个ArrayList,其存储类型是TypeAdapterFactory,并在初始化的时候添加了各种类型的TypeAdapterFactory。

```java
public interface TypeAdapterFactory {
  <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type);
}

```

Gson中其实充斥了大量的TypeAdapterFactory,包基本类型的、用户自定义的等等。

我们先看下基本类型的TypeAdapters.STRING_FACTORY。

```java
  public static final TypeAdapterFactory STRING_FACTORY = newFactory(String.class, STRING);

  public static <TT> TypeAdapterFactory newFactory(
    final Class<TT> type, final TypeAdapter<TT> typeAdapter) {
  return new TypeAdapterFactory() {
    
    @Override public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> typeToken) {
      return typeToken.getRawType() == type ? (TypeAdapter<T>) typeAdapter : null;
    }
    ...
    //省略非关键代码
  };
}

public static final TypeAdapter<String> STRING = new TypeAdapter<String>() {
  @Override
  public String read(JsonReader in) throws IOException {
    JsonToken peek = in.peek();
    if (peek == JsonToken.NULL) {
      in.nextNull();
      return null;
    }
    /* coerce booleans to strings for backwards compatibility */
    if (peek == JsonToken.BOOLEAN) {
      return Boolean.toString(in.nextBoolean());
    }
    return in.nextString();
  }
  @Override
  public void write(JsonWriter out, String value) throws IOException {
    out.value(value);
  }
};

```

## 总结

如果解析一个String类型的数据，大概的逻辑会是这样的：  

- Gson初始化时通过newFactory方法将String类型的TypeAdapter包装成TypeAdapterFactory。
- 开始解析时会在上文的提到的 `getAdaapter()`中获取该TypeAdapterFactory。
- 再调用create方法得到String类型的TypeAdapter。
- 最后调用TypeAdapter的read方法将json转化为String。

好像有哪里不对劲...  

为什么不直接去查找TypeAdapter了？为什么中间要加个TypeAdapterFactory了？这不是脱裤子放屁吗？好问题，我现在也不知道，欲知后事如何，请看下回分解。

看到这里我们终于看到了read方法了，TypeAdapter的read方法作用就是将Json转化为Java Object；write方法时将Java Object转化为Json。但是我们使用Gson时传入的是JavaBean，不是基本类型，这时该怎么获取TypeAdapter去解析json了？欲知后事如何，请看下回分解。



<h3 id="1"></h3>

## TypeToken

TypeToken是用来获取泛型的参数类型的。

```kotlin
data class User(val id: Long, val name: String)
...
//返回List<User>类型
val type = object :TypeToken<List<User>>(){}.type
//返回List类型
val rawType = object :TypeToken<List<User>>(){}.rawType
```
### 那么它的内部是怎么实现的了？

```java
public class TypeToken<T> {
  final Class<? super T> rawType;
  final Type type;
  final int hashCode;


  @SuppressWarnings("unchecked")
  protected TypeToken() {
    this.type = getSuperclassTypeParameter(getClass());
    this.rawType = (Class<? super T>) $Gson$Types.getRawType(type);
    this.hashCode = type.hashCode();
  }

  ...
}

```
这个类的无参构造函数是protected，所以只能使用继承的方式去创建一个实例。构造函数第一行调用了getSuperclassTypeParameter，我们定位到此方法。

```java
  static Type getSuperclassTypeParameter(Class<?> subclass) {
    Type superclass = subclass.getGenericSuperclass();
	 //如果返回的是Class类型则抛出错误
    if (superclass instanceof Class) {
      throw new RuntimeException("Missing type parameter.");
    }
    ParameterizedType parameterized = (ParameterizedType) superclass;
    return $Gson$Types.canonicalize(parameterized.getActualTypeArguments()[0]);
  }
```
关注getGenericSuperclass()这个方法，它可以返回直接继承的父类的参数化类型（包含泛型参数）。

### 参数化类型 ParameterizedType

```java
public interface ParameterizedType extends Type {

    //返回确切的泛型参数, 如Map<String, Integer>返回[String, Integer]
    Type[] getActualTypeArguments();

    //返回当前class或interface声明的类型, 如List<?>返回List
    Type getRawType();

    //返回所属类型. 这主要是对嵌套定义的内部类而言的，例如于对Map.Entry<K,V>来说，调用getOwnerType方法返回的就是Map。
    Type getOwnerType();
}

```

再回过头看一看TypeToken的使用。  

`val type = object :TypeToken<List<User>>(){}.type`  

通过匿名类的形式创建实例，此时实例的父类型就是TypeToken\<List\<User\>\>,所以在构造函数中调用getGenericSuperclass()便可以得到TypeToken\<List\<User\>\>的参数化类型，从而得到List\<User\>类型信息。

#### 疑惑
我们都知道java的泛型擦除机制的存在，那么为什么ParameterizedType可以获取泛型类型了?它是怎么去拿到这个类型信息的了？求大佬解惑。









