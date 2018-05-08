## 核心
- Spring Data Repository抽象，是数据访问操作的统一标准
```
Repository，接受domain类和domain类的ID类型作为类型操作
  - CrudRepository定义了CRUD操作相关的内容
    - PagingAndSortingRepository定义了与分页排序操作相关的内容
      - JpaRepository
      - MongoRepository
```

## Spring Data JPA
- JPA Java Persistence API，主要有Hibernate OpenJPA等实现

## 声明式事务
- Spring Boot自动配置了DataSourceTransactionManager，DataSourceTransactionManager还开启了声明式事务的支持，所以无需配置
- 1.配置类上声明@EnableTransactionManagement
- 2.方法和类上声明@Transactional
  - propagaion 声明事务的声明周期 REQUIRED REQUIRES_NEW NESTED SUPPORTS NOT_SUPPORTED NEVER MANDATORY
  - isolation 声明事务的完整性 READ_UNCOMMITED READ_COMMITTED REPEATABLE_READ SERIALIZABLE DEFAULT
  - timeout
  - readOnly
  - rollbackFor
  - noRollbackFor
