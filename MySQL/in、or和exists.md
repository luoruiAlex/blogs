## in和exists
- in把外表和内表作hash连接
- exists对外表作loop循环，每次loop循环一次对内表进行一次查询
- 如果查询的两个表大小相当，那么用in和exists差别不大
- 两个表中一个较小一个较大
  - 子查询表大的用exists
  - 子查询表小的用in
  
## 实例
- A(5条记录)B(10万条记录)，主键都是id
- select * from A where id in (select id from B)
  - 相当于 for select id from B (for select * from A where A.id = B.id)
- select * from A where exists (select 1 from B where B.id = A.id)
  - 相当于 for select * from A (for select * from B where B.id = A.id)
- 当A表的数据集小于B表的数据集时，用exists优于in
- not in 和not exists如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。

## in和or
- 有索引的时候性能差不多
- 没有索引时，or随记录的增多而性能下降得很快
