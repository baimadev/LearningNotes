# Retrofit

## create
使用动态代理去创建ApiService

```java

public <T> T create(final Class<T> service) {
  validateServiceInterface(service);
  return (T)
      Proxy.newProxyInstance(
          service.getClassLoader(),
          new Class<?>[] {service},
          new InvocationHandler() {
            private final Platform platform = Platform.get();
            private final Object[] emptyArgs = new Object[0];

            @Override
            public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
                throws Throwable {
              // If the method is a method from Object then defer to normal invocation.
              if (method.getDeclaringClass() == Object.class) {
                return method.invoke(this, args);
              }
              args = args != null ? args : emptyArgs;
              return platform.isDefaultMethod(method)
                  ? platform.invokeDefaultMethod(method, service, proxy, args)
                  : loadServiceMethod(method).invoke(args);
            }
          });
}

```

## loadServiceMethod

1.先从缓存获取ServiceMethod。
2.没有就重新创建（因为ServiceMethod使用到了反射）。
3.将新创建的放入缓存。

```java
ServiceMethod<?> loadServiceMethod(Method method) {
  ServiceMethod<?> result = serviceMethodCache.get(method);
  if (result != null) return result;

  synchronized (serviceMethodCache) {
    result = serviceMethodCache.get(method);
    if (result == null) {
      result = ServiceMethod.parseAnnotations(this, method);
      serviceMethodCache.put(method, result);
    }
  }
  return result;
}
```

## parseAnnotations

```java

static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
  RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);

  Type returnType = method.getGenericReturnType();
  if (Utils.hasUnresolvableType(returnType)) {
    throw methodError(
        method,
        "Method return type must not include a type variable or wildcard: %s",
        returnType);
  }
  if (returnType == void.class) {
    throw methodError(method, "Service methods cannot return void.");
  }

  return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
}
```

### RequestFactory.parseAnnotations

解析注解信息，存储在Factory中，方法注解和参数注解信息.

### HttpServiceMethod.parseAnnotations

1.获取returnType；
2.创建CallAdapter；
3.创建Convert（Gson、moshi）；
4.将以上内容组装在适配器中。（适配器继承了HttpServiceMethod）

## HttpServiceMethod执行invoke

1.创建OkHttpCall;
2.执行适配器adapt；

## CallAdapter发起请求

1.根据ApiService返回类型，将OkHttpCall装配在不同的类型中。（Response、Completeable、flow等）；
2.okhttp执行enqueue。
