<h1 align="center">Retrofit</h1>

[toc]

Retrofit与Okhttp的组合也算是业内通用的解决方案了，准备来说Retrofit不是一个网络客户端，而是一个针对Okhttp的网络封装库，让Okhttp的使用更加简便。

> A type-safe HTTP client for Android and Java

官方网站：https://github.com/square/retrofit

源码版本：2.3.0

在分析Retrofit源码实现之前，我们先来看一下它的简单用法。

1. 定义一个Api。

```java
public interface Api {
    @GET("users/{user}/repos")
    Call<List<Repo>> listRepos(@Path("user") String user);
}
```

2. 创建Retrofit实例。

```java
Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://api.github.com/")
            .build();
```

3. 创建Api实例。

```java
Api api = retrofit.create(Api.class);
```

4. 发起网络请求。

```java
Call<List<Repo>> repos = api.listRepos("octocat");
```

你可以发现，Retrofit用注解来标识不同的网络请求类型，这极大的简化了Okhttp的使用方法。了解了上述的使用流程，我们不难发现以下三个问题：

1. Retrofit实例是如何创建的，它初始化了哪些东西？
2。Api实例是如何创建的，这些注解是如何映射到每种网络请求上的？
3. 网络请求是如何发出的？

我们一一来看一下、

## 一、Retrofit创建流程

Retrofits是通过Builder模式来构建的，它主要初始化了以下成员变量：

```java
public final class Retrofit {
    // 网络请求工厂，用来构建网络请求Call
    final okhttp3.Call.Factory callFactory;
    // host url
    final HttpUrl baseUrl;
    // 网络请求解析器，通常是解析JSON，例如：GsonConverterFactory
    final List<Converter.Factory> converterFactories;
    final List<CallAdapter.Factory> adapterFactories;
    final @Nullable Executor callbackExecutor;
    // 是否启用动态代理类的缓存找机会
    final boolean validateEagerly;  
}
```
## 二、Api创建流程

Api的创建是由Retrofit的create()方法完成的，如下所示：

```java
public final class Retrofit {
    
    public <T> T create(final Class<T> service) {
      // 对参数service进行校验，service必须是一个接口，而且没有继承别的接口
      Utils.validateServiceInterface(service);
      if (validateEagerly) {
        eagerlyValidateMethods(service);
      }
      // 利用动态代理技术，自动生成Api接口的实现类，将Api接口方法里参数都交由InvocationHandler来处理。
      return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
          new InvocationHandler() {
            private final Platform platform = Platform.get();
  
            @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
                throws Throwable {
              // Object里的方法不做处理，直接调用它自己的方法。
              if (method.getDeclaringClass() == Object.class) {
                return method.invoke(this, args);
              }
              // 这个isDefaultMethod()方法默认返回false。
              if (platform.isDefaultMethod(method)) {
                return platform.invokeDefaultMethod(method, service, proxy, args);
              }
              // 获取ServiceMethod对象，
              ServiceMethod<Object, Object> serviceMethod =
                  (ServiceMethod<Object, Object>) loadServiceMethod(method);
              OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
              return serviceMethod.callAdapter.adapt(okHttpCall);
            }
          });
    }
   
}
```

> ServiceMethod 就是Api接口对应的正在的方法了，当然它是一个类，里面封装了一次请求的baseUrl、httpMethod、headers等一次网络请求的所有信息。

我们来看看它是如何获取的，如下所示：


```java
public final class Retrofit {
    
     ServiceMethod<?, ?> loadServiceMethod(Method method) {
       // 1. 先从缓存里获取，如果有则直接返回。
       ServiceMethod<?, ?> result = serviceMethodCache.get(method);
       if (result != null) return result;
   
       synchronized (serviceMethodCache) {
         // 2. 这里再同步代码块里又获取了一次，这是因为网络请求一般都在多线程里执行，这个时候，可能
         // 又创建完成了。
         result = serviceMethodCache.get(method);
         if (result == null) {
           // 3. 调用ServiceMethod的Builder类进行构建。
           result = new ServiceMethod.Builder<>(this, method).build();
           // 4. 存入缓存。
           serviceMethodCache.put(method, result);
         }
       }
       return result;
     }
}
```

该方法的执行逻辑如下所示：

1. 先从缓存里获取，如果有则直接返回。
2. 这里再同步代码块里又获取了一次，这是因为网络请求一般都在多线程里执行，这个时候，可能又创建完成了。
3. 调用ServiceMethod的Builder类进行构建。
4. 存入缓存。

我们可以看看ServiceMethod里包含了哪些信息，它主要包含了这次请求的网络信息，如下所示：

```java
final class ServiceMethod<R, T> {
    
      final okhttp3.Call.Factory callFactory;
      final CallAdapter<R, T> callAdapter;
    
      private final HttpUrl baseUrl;
      private final Converter<ResponseBody, R> responseConverter;
      private final String httpMethod;
      private final String relativeUrl;
      private final Headers headers;
      private final MediaType contentType;
      private final boolean hasBody;
      private final boolean isFormEncoded;
      private final boolean isMultipart;
      private final ParameterHandler<?>[] parameterHandlers;

}
```

这里面除了网络相关信息，还包含了：

- okhttp3.Call.Factory callFactory：负责创建 HTTP 请求，HTTP 请求被抽象为了 okhttp3.Call 类，它表示一个已经准备好，可以随时执行的 HTTP 请求；
- CallAdapter<R, T> callAdapter：把 retrofit2.Call<T> 转为 T（注意和 okhttp3.Call 区分开来，retrofit2.Call<T> 表示的是对一个 Retrofit 方法的调用），这
个过程会发送一个 HTTP 请求，拿到服务器返回的数据（通过 okhttp3.Call 实现），并把数据转换为声明的 T 类型对象（通过 Converter<F, T> 实现）；
- Converter<ResponseBody, R> responseConverter：负责把服务器返回的数据（JSON、XML、二进制或者其他格式，由 ResponseBody 封装）转化为 T 类型的对象；
- ParameterHandler<?>[] parameterHandlers 注解解析器，负责解析 API 定义时每个方法的参数，并在构造 HTTP 请求时设置参数；

关于retrofit-adapters模块

> retrofit 模块内置了 DefaultCallAdapterFactory 和 ExecutorCallAdapterFactory，它们都适用于 API 方法得到的类型为 retrofit2.Call 的情形，前者生产的 adapter 啥也不做，直接把参数返回，后者生产的 adapter 则会在异步调用时在指定的 Executor 上执行回调。

关于retrofit-converters模块

> retrofit 模块内置了 BuiltInConverters，只能处理 ResponseBody， RequestBody 和 String 类型的转化（实际上不需要转）。而 retrofit-converters 中的子模块则提供了 JSON，XML，ProtoBuf 等类型数据的转换功能，而且还有多种转换方式可以选择。这里我主要关注 GsonConverterFactory。

注：上述提供了三种工厂：okhttp3.Call.Factory、CallAdapter.Factory与Coverter.Factory，它将网络请求、请求处理与返回解析完全解耦，这些工厂都由外部提供，Retrofit
本身并不参与，它只是负责提供一些参数供它们进行决策。

这也给我们解耦提供了一种非常好的思路，解耦的第一步是面向接口编程，各模块之前通过接口进行依赖，具体创建什么样的实例由工厂方法来负责，主类（例如：Retrofit）只是负责提供相关参数
以及参数的处理。

## 三、网络请求的发送流程

请求是由OkHttpCall类来完成的，它实现了Call接口，它同样包含同步请求和异步请求两种。

### 3.1 同步请求

````java
final class OkHttpCall<T> implements Call<T> {
    
    @Override public Response<T> execute() throws IOException {
        okhttp3.Call call;
    
        // 同步
        synchronized (this) {
          if (executed) throw new IllegalStateException("Already executed.");
          executed = true;
    
          if (creationFailure != null) {
            if (creationFailure instanceof IOException) {
              throw (IOException) creationFailure;
            } else {
              throw (RuntimeException) creationFailure;
            }
          }
    
          call = rawCall;
          if (call == null) {
            try {
              // 1. 创建Call对象。
              call = rawCall = createRawCall();
            } catch (IOException | RuntimeException e) {
              creationFailure = e;
              throw e;
            }
          }
        }
    
        if (canceled) {
          call.cancel();
        }
    
        // 2. 执行请求并解析返回结果。
        return parseResponse(call.execute());
      }  
}
````

同步请求的流程是十分简单的，如下所示：

1. 创建Call对象。
2. 执行请求并解析返回结果。


### 3.2 异步请求


```java
final class OkHttpCall<T> implements Call<T> {
    
    @Override public void enqueue(final Callback<T> callback) {
       checkNotNull(callback, "callback == null");
   
       okhttp3.Call call;
       Throwable failure;
   
       synchronized (this) {
         if (executed) throw new IllegalStateException("Already executed.");
         executed = true;
   
         call = rawCall;
         failure = creationFailure;
         if (call == null && failure == null) {
           try {
               
             // 1. 创建Call对象。
             call = rawCall = createRawCall();
           } catch (Throwable t) {
             failure = creationFailure = t;
           }
         }
       }
   
       if (failure != null) {
         callback.onFailure(this, failure);
         return;
       }
   
       if (canceled) {
         call.cancel();
       }
       // 2. 调用Call的异步执行方法。
       call.enqueue(new okhttp3.Callback() {
         @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
             throws IOException {
           Response<T> response;
           try {
             // 3. 解析返回结果。
             response = parseResponse(rawResponse);
           } catch (Throwable e) {
             callFailure(e);
             return;
           }
           // 5. 执行回调。
           callSuccess(response);
         }
   
         @Override public void onFailure(okhttp3.Call call, IOException e) {
           try {
             callback.onFailure(OkHttpCall.this, e);
           } catch (Throwable t) {
             t.printStackTrace();
           }
         }
   
         private void callFailure(Throwable e) {
           try {
             callback.onFailure(OkHttpCall.this, e);
           } catch (Throwable t) {
             t.printStackTrace();
           }
         }
   
         private void callSuccess(Response<T> response) {
           try {
             callback.onResponse(OkHttpCall.this, response);
           } catch (Throwable t) {
             t.printStackTrace();
           }
         }
       });
     } 
}
```
异步请求的流程如下所示：

1. 创建Call对象。
2. 调用Call的异步执行方法。
3. 解析返回结果。
5. 执行回调。