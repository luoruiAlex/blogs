## 核心
- CacheManager
- Cache(一般不直接和这个接口打交道)

## 支持的CacheManager
- SimpleCacheManager 用collection来存储，用于测试
- EhCacheCacheManager
- GuavaCacheManager
- RedisCacheManager

## 配置
- @Bean 返回CacheManager的实现类的Bean，可注册
- @EnableCaching 开启声明式缓存支持
- Spring Boot自动配置了多个CacheManager的实现，比如RedisCacheConfiguration，只需导入依赖包并使用@EnableCaching即可
  - EhCache的配置文件ehcache.xml放到类路径下即可
  - Guava：自动配置GuavaCacheManager
  - Redis：自动配置RedisCacheManager及RedisTemplate的Bean
