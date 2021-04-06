#Gson源码解析

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




### 疑惑
大家都知道java的泛型擦除机制的存在，那么为什么Gson可以在运行时拿到具体的泛型类型了？

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();
		
System.out.println(l1.getClass() == l2.getClass());
```

回想刚开始学java时看到的博客 “为了兼容jdk1.5之前的版本，java会在编译期擦除与泛型相关的信息。”  
这句话是对的，但没有完全对，有被误导到...

https://techblog.bozho.net/on-java-generics-and-erasure/  
https://zhuanlan.zhihu.com/p/292983882  

参考了上面两篇博客了解到编译期并不会完全擦除泛型信息：

- 1、泛型不止在编译阶段生效，部分泛型可以在运行时通过反射获取；  
- 2、java 语言尝试将所有能确定的泛型信息记录在类文件中。  


### 未被擦除的泛型

父类泛型、成员变量、方法入参和返回值使用到的泛型信息都会保留，并能在运行阶段获取。

```java
public static void main(String[] args) throws Exception {
        // 获取父类泛型
        System.out.println("GenericSuperclass:" + ((ParameterizedType) (Clazz.class.getGenericSuperclass())).getActualTypeArguments()[0]);
        // 获取成员变量泛型
        Field field = Clazz.class.getDeclaredField("field");
        for (Type fieldType : ((ParameterizedType) (field.getGenericType())).getActualTypeArguments()) {
            System.out.println("field:" + fieldType.getTypeName());
        }
        // 获取方法入参和返回值泛型
        Method method = Clazz.class.getDeclaredMethod("function", List.class);
        for (Type type : method.getGenericParameterTypes()) {
            System.out.println("method param:" + ((ParameterizedType) type).getActualTypeArguments()[0]);
        }
        System.out.println("method return:" + ((ParameterizedType) method.getGenericReturnType()).getActualTypeArguments()[0]);
    }

```

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


### TypeToken的实现

再回过头看一看TypeToken的实现就简单了。  



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


`val type = object :TypeToken<List<User>>(){}.type`  

通过匿名类的形式创建实例，此时实例的父类型就是TypeToken\<List\<User\>\>。


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

调用getGenericSuperclass()便可以得到TypeToken\<List\<User\>\>的参数化类型，从而得到List\<User\>类型信息。



## 反射机制详解

接着上篇留下来的问题，Gson是怎么序列化我们自己创建的JavaBean对象？

还记得在gson初始化时添加的一系列Factory吗，我们可以看到它添加了一个  
`   factories.add(new ReflectiveTypeAdapterFactory(
       constructorConstructor, fieldNamingStrategy, excluder, jsonAdapterFactory));
`  
看名字我们应该也可以猜出来这个Factory的作用，没错，它可以根据用户传入的类型，通过反射创建出该类型的实例。  

我们定位到它的create方法。

```java
  @Override public <T> TypeAdapter<T> create(Gson gson, final TypeToken<T> type) {
    Class<? super T> raw = type.getRawType();

    if (!Object.class.isAssignableFrom(raw)) {
      return null; // it's a primitive!
    }

    ObjectConstructor<T> constructor = constructorConstructor.get(type);
    return new Adapter<T>(constructor, getBoundFields(gson, type, raw));
  }
```

创建了一个TypeAdapter并返回，我们先不看constructor和getBoundFields(),直接定位到Adapter的read方法。

```java
public static final class Adapter<T> extends TypeAdapter<T> {
    private final ObjectConstructor<T> constructor;
    private final Map<String, BoundField> boundFields;

    Adapter(ObjectConstructor<T> constructor, Map<String, BoundField> boundFields) {
      this.constructor = constructor;
      this.boundFields = boundFields;
    }

    @Override public T read(JsonReader in) throws IOException {
      if (in.peek() == JsonToken.NULL) {
        in.nextNull();
        return null;
      }

      T instance = constructor.construct();

      try {
        in.beginObject();
        while (in.hasNext()) {
          String name = in.nextName();
          BoundField field = boundFields.get(name);
          if (field == null || !field.deserialized) {
            in.skipValue();
          } else {
            field.read(in, instance);
          }
        }
      } catch (IllegalStateException e) {
        throw new JsonSyntaxException(e);
      } catch (IllegalAccessException e) {
        throw new AssertionError(e);
      }
      in.endObject();
      return instance;
    }
 }

```

ok看到这里我们大致能猜到constructor和BoundField是做什么的。

- 通过constructor创建实例对象
- 通过BoundField获取Field和value并设置给实例对象

接下来我们看看它们的具体实现。

## ConstructorConstructor

我们来看看ConstructorConstructor的get方法：

```java

  public <T> ObjectConstructor<T> get(TypeToken<T> typeToken) {
    final Type type = typeToken.getType();
    final Class<? super T> rawType = typeToken.getRawType();

    // first try an instance creator
    @SuppressWarnings("unchecked") // types must agree
    final InstanceCreator<T> typeCreator = (InstanceCreator<T>) instanceCreators.get(type);
    if (typeCreator != null) {
      return new ObjectConstructor<T>() {
        @Override public T construct() {
          return typeCreator.createInstance(type);
        }
      };
    }

    // Next try raw type match for instance creators
    @SuppressWarnings("unchecked") // types must agree
    final InstanceCreator<T> rawTypeCreator =
        (InstanceCreator<T>) instanceCreators.get(rawType);
    if (rawTypeCreator != null) {
      return new ObjectConstructor<T>() {
        @Override public T construct() {
          return rawTypeCreator.createInstance(type);
        }
      };
    }

    ObjectConstructor<T> defaultConstructor = newDefaultConstructor(rawType);
    if (defaultConstructor != null) {
      return defaultConstructor;
    }

    ObjectConstructor<T> defaultImplementation = newDefaultImplementationConstructor(type, rawType);
    if (defaultImplementation != null) {
      return defaultImplementation;
    }

    // finally try unsafe
    return newUnsafeAllocator(type, rawType);
  }
```
代码有点长，但看起来就很舒服，这里它采用了策略模式，通过四种方法去获得ObjectConstructor，在通过ObjectConstructor.construct()方法获得对象实例。

1.首先尝试通过instanceCreateor获得ObjectConstructor。我们可以通过registerTypeAdapter的方式创建与type对应的instanceCreateor。

```kotlin
       val gson2 = GsonBuilder()
            .registerTypeAdapter(User::class.java, InstanceCreator {
                    User()//直接创建个实例
            })
            .create()
```

2.如果找不到与type对应的instanceCreateor就调用newDefaultConstructor(type).

```java
  private <T> ObjectConstructor<T> newDefaultConstructor(Class<? super T> rawType) {
    
      final Constructor<? super T> constructor = rawType.getDeclaredConstructor();
      if (!constructor.isAccessible()) {
        accessor.makeAccessible(constructor);
      }
      return new ObjectConstructor<T>() {
    
            Object[] args = null;
            return (T) constructor.newInstance(args);
		
		//...省略非关键代码
   }
  }

```
这下就很明了了，Gson序列化就是通过反射去创建的实例对象。

3.newDefaultImplementationConstructor()是为Map、List、Set等等准备的constructor。

```java
  /**
   * Constructors for common interface types like Map and List and their
   * subtypes.
   */
  @SuppressWarnings("unchecked") // use runtime checks to guarantee that 'T' is what it is
  private <T> ObjectConstructor<T> newDefaultImplementationConstructor(
      final Type type, Class<? super T> rawType) {
    if (Collection.class.isAssignableFrom(rawType)) {
      if (SortedSet.class.isAssignableFrom(rawType)) {
        return new ObjectConstructor<T>() {
          @Override public T construct() {
            return (T) new TreeSet<Object>();
          }
        };
      } else if (EnumSet.class.isAssignableFrom(rawType)) {
        return new ObjectConstructor<T>() {
          @SuppressWarnings("rawtypes")
          @Override public T construct() {
            if (type instanceof ParameterizedType) {
              Type elementType = ((ParameterizedType) type).getActualTypeArguments()[0];
              if (elementType instanceof Class) {
                return (T) EnumSet.noneOf((Class)elementType);
              } else {
                throw new JsonIOException("Invalid EnumSet type: " + type.toString());
              }
            } else {
              throw new JsonIOException("Invalid EnumSet type: " + type.toString());
            }
          }
        };
      } else if (Set.class.isAssignableFrom(rawType)) {
        return new ObjectConstructor<T>() {
          @Override public T construct() {
            return (T) new LinkedHashSet<Object>();
          }
        };
      } else if (Queue.class.isAssignableFrom(rawType)) {
        return new ObjectConstructor<T>() {
          @Override public T construct() {
            return (T) new ArrayDeque<Object>();
          }
        };
      } else {
        return new ObjectConstructor<T>() {
          @Override public T construct() {
            return (T) new ArrayList<Object>();
          }
        };
      }
    }

```

4.最终会调用ConstructorConstructor类中的newUnsafeAllocator方法来为你创建对应的JavaBean对象，如果此时创建失败的话就会抛出异常。


## BoundField


```java
  private Map<String, BoundField> getBoundFields(Gson context, TypeToken<?> type, Class<?> raw) {
    Map<String, BoundField> result = new LinkedHashMap<String, BoundField>();
    if (raw.isInterface()) {
      return result; //如果是接口直接返回
    }

    Type declaredType = type.getType();
    while (raw != Object.class) {
      Field[] fields = raw.getDeclaredFields();
      for (Field field : fields) {
        boolean serialize = excludeField(field, true);
        boolean deserialize = excludeField(field, false);
		  //如果不能序列化和反序列化就直接跳过
        if (!serialize && !deserialize) {
          continue;
        }
        accessor.makeAccessible(field);
        Type fieldType = $Gson$Types.resolve(type.getType(), raw, field.getGenericType());
        //获取Field的Name，之所以是个数组，
        //是因为我么可以通过@SerializedName("a",alternate = ["b","c"])添加字段别名
        List<String> fieldNames = getFieldNames(field);
        BoundField previous = null;
        for (int i = 0, size = fieldNames.size(); i < size; ++i) {
          String name = fieldNames.get(i);
          if (i != 0) serialize = false; // only serialize the default name
          //创建BoundField
          BoundField boundField = createBoundField(context, field, name,
              TypeToken.get(fieldType), serialize, deserialize);
          
          BoundField replaced = result.put(name, boundField);
          if (previous == null) previous = replaced;
        }
        if (previous != null) {
          throw new IllegalArgumentException(declaredType
              + " declares multiple JSON fields named " + previous.name);
        }
      }
      type = TypeToken.get($Gson$Types.resolve(type.getType(), raw, raw.getGenericSuperclass()));
      raw = type.getRawType();
    }
    return result;
  }

```
BoundField是ReflectTypeAdapterFactory的一个内部抽象类。

```java
  static abstract class BoundField {
    final String name;
    final boolean serialized;
    final boolean deserialized;

    protected BoundField(String name, boolean serialized, boolean deserialized) {
      this.name = name;
      this.serialized = serialized;
      this.deserialized = deserialized;
    }
    abstract boolean writeField(Object value) throws IOException, IllegalAccessException;
    abstract void write(JsonWriter writer, Object value) throws IOException, IllegalAccessException;
    abstract void read(JsonReader reader, Object value) throws IOException, IllegalAccessException;
  }


```

我们定位到createBoundField方法，这里省略了write方法。

```java
private ReflectiveTypeAdapterFactory.BoundField createBoundField(
      final Gson context, final Field field, final String name,
      final TypeToken<?> fieldType, boolean serialize, boolean deserialize) {
    final boolean isPrimitive = Primitives.isPrimitive(fieldType.getRawType());
    // special casing primitives here saves ~5% on Android...
    JsonAdapter annotation = field.getAnnotation(JsonAdapter.class);
    TypeAdapter<?> mapped = null;
    //如果这个字段有使用到JsonAdapter注解，另外处理
    if (annotation != null) {
      mapped = jsonAdapterFactory.getTypeAdapter(
          constructorConstructor, context, fieldType, annotation);
    }
    final boolean jsonAdapterPresent = mapped != null;
    //熟悉的代码。调用gson.getAdapter()
    if (mapped == null) mapped = context.getAdapter(fieldType);

    final TypeAdapter<?> typeAdapter = mapped;
    return new ReflectiveTypeAdapterFactory.BoundField(name, serialize, deserialize) {
      ...
      @Override void read(JsonReader reader, Object value)
          throws IOException, IllegalAccessException {
        //调用了TypeAdapter的read方法获取fieldValue
        Object fieldValue = typeAdapter.read(reader);
        //将fieldValue设置给传过来的实例对象
        if (fieldValue != null || !isPrimitive) {
          field.set(value, fieldValue);
        }
      }
		...
  }

```
主要逻辑就是通过fieldType获取TypeAdapter，在通过TypeAdapter.read()读取Json对应filed的具体值，然后通过反射将值设置给我们传进来的实例对象。

如果这个Field本身就是个JavaBean对象了，也是没问题的，通过gson.getAdapter()它会得到对应的TypeAdapter，然后再去通过反射创将对象、设值......

到这里整个反射机制流程大致的介绍完毕了。





