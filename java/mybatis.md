## 对比
### Hibernate
- 优点
  - 不需要管理数据库连接(在配置文件中)
  - 一个会话中，不要操作多个对象，只需操作Session对象即可
  - 关闭资源只需关闭Session即可
- 缺点
  - 全表映射带来不便，比如更新时需要发送所有字段
  - 无法根据不同条件组装不同SQL
  - 多表关联和复杂SQL查询支持较差，需要自己组装数据
  - 不能有效支持存储过程
  - 性能差
### MyBatis
- 灵活，可以动态生成映射关系
- 拥有动态列、动态表名
- 支持存储过程
- 支持缓存、日志、级联
- 需要提供映射规则和SQL，开发工作量比Hibernate略大

## 插入后返回主键
- 修改<insert>标签的属性
  - useGeneratedKeys="true"(给主键设置自增长)
  - keyProperty="userId"(表示将自增长后的Id赋值给实体类中的userId字段)
- <insert>标签中增加<selectKey>标签 
  - resultType="java.lang.Long"
  - order="AFTER"
  - keyProperty="productId"
  - 语句为SELECT LAST_INSERT_ID()
- 注意，插入没有返回值
  
## select
- resultType与resultMap二选一
- flushCache="true"的语句调用后会导致本地缓存和二级缓存被清空
- useCache="true"使本语句的结构被二级缓存
- statementType="PREPARED"默认为PREPARED，也可为STATEMENT或者CALLABLE
- timeout、fetchSize、resultSetType 默认为unset

## ResultMap
- type、id
- collection
```
<collection property="courseList" column="stu_course_id" ofType="Course">
    <id property="id" column="course_id"/>
    <result property="name" column="course_name"/>
    <result property="deleteFlag" column="course_delete_flg"/>
</collection>
```
- select resultMap="mapId"
```
SELECT s.*, c.* FROM t_student s LEFT JOIN t_course c ON s.stu_course_id=c.course_id WHERE s.stu_id_card=#{idCard}
```
