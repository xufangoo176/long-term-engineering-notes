# Guava Cache 的基本介绍

Guava Cache 作为本地缓存，速度是非常快的，比 Redis 这种分布式缓存要快很多，因为 Redis 还要走网络去进行请求和响应，而本地缓存则无需这种开销。



对了，Guava 是 Google 公司开源的 Java 工具库，提供的是 JDK 没有的功能，还有增强 JDK 的功能。

比如：

- `com.google.common.collect`: 集合工具包，提供了许多 JDK 中没有的集合类型和集合操作方法。
- `com.google.common.io`: I/O 工具包，提供了许多实用的 I/O 操作类和工具方法。
- `com.google.common.cache`: 缓存工具包，提供了一个高性能的本地缓存实现。
- `com.google.common.math`: 数学工具包，提供了许多数学计算和运算的实用方法。
- `com.google.common.eventbus`: 事件总线工具包，提供了基于观察者模式的事件发布和订阅功能。
- `com.google.common.reflect`: 反射工具包，提供了更好用的反射 API。
- `com.google.common.util.concurrent`: 并发工具包，提供了许多 JDK 并发包中没有的并发实用工具类和工具方法。

可以看得出，Guava 非常的强大，Cache 只是其中的一部分功能而已。

那如何在 Spring Boot 应用中整合 Guava Cache 呢？早期版本有两种方式，一种是使用

`GuavaCacheManager`，一种是使用 `CacheBuilder`。

注意：Spring Boot 版本超过 2.7.1，只能使用 `CacheBuilder` 的方式。



# 使用 GuavaCacheManager

在 Spring Boot 早期版本中，集成 Guava Cache 非常的简单，因为 Spring 定义了 CacheManager 接口来统一

不同的缓存技术，比如说 Guava、Redis、Caffeine。

- GuavaCacheManager: 使用 Guava 作为缓存技术
- RedisCacheManager: 使用 Redis 作为缓存技术
- CaffeineCacheManager: 使用 Caffeine 作为缓存技术
- ConcurrentMapCacheManager: Spring Boot 默认的缓存实现

我们重点来说说 GuavaCacheManager，它提供了一种基于注解的缓存管理方式，通过在方法上加上

`@Cacheable`、`@CachePut`、`@CacheEvict` 等注解，实现对方法返回结果的缓存。

- `@Cacheable`: 在方法执行前，Spring 会检查缓存中是否存在相同 key 的缓存数据，如果存在，直接返回缓存数据，如果不存在，则执行方法并将返回结果缓存起来。
- `@CachePut`: 无论缓存中是否已存在相同 key 的缓存数据，都会执行方法并将返回结果缓存起来，用于更新缓存数据。
- `@CacheEvict`: 用于删除指定 key 的缓存数据。

**不过，Spring Boot 2.7.1 版本已经把 GuavaCacheManager 移除了**，取而代之的是 CaffeineCacheManager。

见下图，在 Spring 中已经找不到 GuavaCacheManager 的身影了。

![image-20260118000111159](/Users/xufan/workspace/long-term-engineering-notes/backend/技术/cache（缓存）/本地缓存/image/image-20260118000111159.png)





# 使用 CacheBuilder

1、在 pom.xml 中添加依赖

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
</dependency>
```

2 、使用

```java
// 创建一个 CacheBuilder 对象
CacheBuilder<Object, Object> cacheBuilder = CacheBuilder.newBuilder()
        .maximumSize(100)  // 最大缓存条目数
        .expireAfterAccess(30, TimeUnit.MINUTES) // 缓存项在指定时间内没有被访问就过期
        .recordStats();  // 开启统计功能

// 构建一个 LoadingCache 对象
LoadingCache<String, String> cache = cacheBuilder.build(new CacheLoader<String, String>() {
    @Override
    public String load(String key) throws Exception {
        return "value: " + key; // 当缓存中没有值时，加载对应的值并返回
    }
});

// 存入缓存
cache.put("name", "徐帆");

// 从缓存中获取值
// put 过
System.out.println(cache.get("name"));
// 没 put 过
System.out.println(cache.get("paicoding"));

// 打印缓存的命中率等统计信息
System.out.println(cache.stats());
```

**具体说明**

CacheBuilder 是 Guava Cache 中用于构建本地缓存的构造器，通过 CacheBuilder 可以配置缓存的各种属性，如最大缓存项数量、缓存项过期时间、缓存项移除通知等。



常用的 CacheBuilder 方法

1. **newBuilder()**

   该方法返回一个 `CacheBuilder` 实例，用于创建一个新的缓存实例。

2. **maximumSize(long maximumSize)**

   该方法用于设置缓存的最大大小，以条目数为单位。如果缓存中的条目数超过了最大大小，则可能会触发缓存的回收策略，以释放一些缓存空间。

3. **expireAfterWrite(long duration, TimeUnit unit)**

   该方法用于设置缓存的过期时间。在缓存中存储的每个条目被创建或者更新后，经过指定的时间后，该条目将被自动删除。可以使用 `TimeUnit` 枚举类型中的常量来指定时间单位，比如 `TimeUnit.SECONDS`、`TimeUnit.MINUTES` 等。

4. **expireAfterAccess(long duration, TimeUnit unit)**

   该方法和 `expireAfterWrite` 方法类似，不同的是，该方法用于设置缓存中每个条目的最大闲置时间。如果一个条目在指定的时间内没有被访问，则该条目将被自动删除。

5. **stats()**

   该方法用于启用缓存统计功能，可以用于监控缓存的使用情况。

6. **build()**

   该方法用于创建和返回一个新的缓存实例。在调用该方法之前，需要进行一些其他的配置，比如设置缓存的最大大小、过期时间等。



`LoadingCache` 是一种特殊的 `Cache`，在缓存中不存在某个 key 的值时，可以通过 `CacheLoader` 来加载该 key 对应的值，并将其加入到缓存中。`LoadingCache` 接口中定义了许多有用的方法，主要包括：

- **get(K key)**：获取指定 key 对应的缓存值。如果缓存中不存在该 key 对应的值，则调用 `CacheLoader` 中的 `load` 方法来加载缓存值。如果 `load` 方法返回 `null`，则 `get` 方法也会返回 `null`。如果 `load` 方法抛出异常，则 `get` 方法会将异常转换为 `ExecutionException`，并将其抛出。

- **put(K key, V value)**：将指定 key 和 value 存入缓存中。如果之前缓存中已经存在该 key 对应的值，则会覆盖之前的缓存值。如果 `value` 为 `null`，则 `put` 方法会抛出 `NullPointerException` 异常。

- **invalidate(K key)**：使指定 key 对应的缓存值失效，并从缓存中移除该 key 对应的值。

- **invalidateAll()**：使所有缓存值失效，并从缓存中移除所有缓存值。

- **size()**：返回缓存中当前存储的 key-value 对数量。

- **stats()**：返回缓存的统计信息，包括缓存命中率、缓存加载成功率、缓存加载平均时间等。

- **ConcurrentMap<K, V> asMap()**：返回一个线程安全的 `ConcurrentMap。

  

`CacheLoader` 是 Guava 缓存库中的一个接口，用于在缓存未命中时加载缓存值。它定义了一个方法 `load(K key)`，当缓存中没有 key 对应的值时，会调用该方法来获取该 key 对应的值并将其存入缓存中。

- load 方法中可以实现从数据源中加载缓存值的逻辑。
- load 方法中可以抛出异常以表示加载失败的情况，例如数据源访问异常等。
- 当使用 LoadingCache 时，load 方法中的实现应该是幂等的，即多次调用应该返回相同的结果。

在上面的示例中，我们在缓存中放了一个键值对（name，徐帆），然后通过 get 方法从缓存中取出，key 为 paicoding 这个在缓存中没有值，所以会通过 CacheLoader 加载一个值并返回。













​	