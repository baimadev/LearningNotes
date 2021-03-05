
#Gson源码解析（二）反射机制详解

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

