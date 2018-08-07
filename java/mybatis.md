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

## 执行流程
- SqlSessionFactoryBuilder读取配置文件，生成 SqlSessionFactory
- SqlSessionFactory获取SqlSession对象

## 配置文件
- configuration节点为根节点
- 10个子节点
- properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers

## 插入后返回主键
- 修改`<insert>`标签的属性
  - useGeneratedKeys="true"(给主键设置自增长)
  - keyProperty="userId"(表示将自增长后的Id赋值给实体类中的userId字段)
- `<insert>`标签中增加`<selectKey>`标签 
  - resultType="java.lang.Long"
  - order="AFTER"
  - keyProperty="productId"
  - 语句为SELECT LAST_INSERT_ID()
- 注意，插入没有返回值

## `#{}`与`${}`的差别
- `#{}`标识一个占位符，向占位符输入参数，mybatis自动进行java类型和jdbc类型的转换，程序员不需要考虑参数的类型，比如传入字符串，mybatis最终拼接好的sql就是参数两边加单引号
- `${}`标识sql的拼接，通过${}接收参数，将参数的内容不加任何修饰拼接在sql中，模糊查询`like`应该用`${}`
  
## select
- resultType与resultMap二选一
- flushCache="true"的语句调用后会导致本地缓存和二级缓存被清空
- useCache="true"使本语句的结构被二级缓存
- statementType="PREPARED"默认为PREPARED，也可为STATEMENT或者CALLABLE
- timeout、fetchSize、resultSetType 默认为unset

## ResultMap
- type、id
- <id>用于表示javavean对象的唯一性，不一定是数据库的逐渐
- collection
```
<collection property="courseList" column="stu_course_id" ofType="Course" javaType="java.util.ArrayList">
    <id property="id" column="course_id"/>
    <result property="name" column="course_name"/>
    <result property="deleteFlag" column="course_delete_flg"/>
</collection>
```
- select resultMap="mapId" fetchType可以为lazy或者eager
```
SELECT s.*, c.* FROM t_student s LEFT JOIN t_course c ON s.stu_course_id=c.course_id WHERE s.stu_id_card=#{idCard}
```
- association与collection的区别在于association用于一对一，而且应用javaType而不是ofType(JavaType是用来指定pojo中属性的类型，而ofType指定的是映射到list集合属性中pojo的类型)
- fetchType="lazy"延迟加载，真正用到的时候才发出SQL查询

## 动态语句
- `<if test="xx">`
- `<foreach item="item" index="index" collection="list" open="(" seperator=", close=")"> #{item}</foreach>`
- `<choose><when></when><otherwise></otherwise></choose>`
- `<where>`标签相当于where，里面包含`<if>`可自动去除第一个and
- sql片段
  - 定义 `<sql id="xxx">`
  - 使用 `<include refid="sqlId">`，如果不在同一个mapper，应加上namespace（namespace.refid）
- `<foreach collection="ids" open="and id in (" close=")" item="id" seperator=",">`

## TypeHandler处理枚举
- EnumTypeHandler: 用于保存枚举名
- EnumOrdinalTypeHandler: 用于保存枚举的序号
- 自定义 extends BaseTypeHandler
  - @MappedJdbcTypes定义的是JdbcType类型，这里的类型不可自己随意定义，必须要是枚举类org.apache.ibatis.type.JdbcType所枚举的数据类型
  - @MappedTypes定义的是JavaType的数据类型，描述了哪些Java类型可被拦截
  - 在setNonNullParameter方法中，我们重新定义要写往数据库的数据，在另外三个方法中我们将从数据库读出的数据类型进行转换
- 注册：`<typeHandlers>`中配置`<typeHandler handler="xxx.xxx.MyDateTypeHandler">`或者`<package name="org.xx">`
- 查询使用 resultMap中`<result typeHandler="xxTypeHandler" />`
- 插入使用 `#{regTime,javaType=Date,jdbcType=VARCHAR,typeHandler=org.sang.db.MyDateTypeHandler}`，可只用typeHandler，或者同时用javaType和jdbcType
