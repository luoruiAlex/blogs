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