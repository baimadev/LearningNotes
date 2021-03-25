
#Gson源码解析（三）TypeToken详解

TypeToken是用来获取泛型的参数类型的。

```kotlin
data class User(val id: Long, val name: String)
...
//返回List<User>类型
val type = object :TypeToken<List<User>>(){}.type
//返回List类型
val rawType = object :TypeToken<List<User>>(){}.rawType
```
## 那么它的内部是怎么实现的了？

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

## 参数化类型 ParameterizedType

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




