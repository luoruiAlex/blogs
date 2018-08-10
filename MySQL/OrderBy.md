## Order By有两种排序实现方式
### 利用有序索引获取有序数据(explain显示Using index)
```
  CREATE TABLE `test` (  
    `id` int(11) NOT NULLAUTO_INCREMENT,  
    `rdate` datetime NOT NULL,  
    `inventid` int(11) NOT NULL,  
    `customerid` int(11) NOT NULL,  
    `staffid` int(11) NOT NULL,  
    `data` varchar(20) NOT NULL,  
    PRIMARY KEY (`id`),  
    UNIQUE KEY `rdate`(`rdate`,`inventid`,`customerid`),  
    KEY `inventid` (`inventid`),  
    KEY `customerid` (`customerid`),  
    KEY `staffid` (`staffid`)  
  ) ENGINE=InnoDB AUTO_INCREMENT=27 DEFAULT CHARSET=latin1  
```
- MySQL在查询时最多只能使用一个索引。因此，如果WHERE条件已经占用了索引，那么在排序中就不使用索引了
- 1.返回选择的字段，即只包括在有选择的此列上（select后面的字段）
  - `explain select inventid from test where rdate='2011-12-1400:00:00' order by  inventid;`
- Select选择的列使用索引则order by不使用索引，以下为Using filesort
  - `explain select * from test where customerid=1  order by  inventid;`
- 2.只有当ORDER BY中所有的列必须包含在相同的索引，并且索引的顺序和order by子句中的顺序完全一致，并且所有列的排序方向（升序或者降序）一样才有，（混合使用ASC模式和DESC模式则不使用索引）
  - `explain select inventid from test order byrdate, inventid;` Using index
  - `explain select inventid from test where rdate="2011-12-16" order by  inventid ,staffid;` Using filesort
- 3. where 语句与ORDER BY语句组合满足最左前缀
  - `explain select inventid from test where rdate="2011-12-16" order by inventid;` Using index
- 4.如果查询联接了多个表，只有在order by子句的所有列引用的是第一个表的列才可以
### 文件排序(explain显示Using filesort)
- 每个线程有一个排序区
- 双路排序：两次I/O
- 单路排序：一次I/O，需要跟多sort buffer。
- 根据 max_length_for_sort_data的大小和Query 语句所取出的字段类型大小总和来判定需要使用哪一种排序算法
- 如果order by的子句只引用了联接中的第一个表，MySQL会先对第一个表进行排序，然后进行联接。也就是expain中的Extra的Using Filesort.否则MySQL先把结果保存到临时表(Temporary Table),然后再对临时表的数据进行排序.此时expain中的Extra的显示Using temporary Using Filesort.
- 优化Filesort
  - 1、尽量让MySQL使用单路排序
    - 加大max_length_for_sort_data
    - 去掉不必要的返回字段
  - 2、增大 sort_buffer_size 参数设置，减少排序过程中对排序的数据进行分段，分段会使MySQL使用临时表来交换排序
